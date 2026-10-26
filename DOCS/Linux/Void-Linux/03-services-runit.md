---
parent: Void Linux
title: 03 Runit
layout: default
---

# Partie 3 : Les services sous Void (runit)

## 3.1 Principe

Void utilise **runit** comme système d'init et superviseur de services (et non systemd). Le fonctionnement est volontairement simple.

Le démarrage se fait en **trois étapes** :

| Étape | Fichier | Rôle |
|---|---|---|
| 1 | `/etc/runit/1` et `/etc/runit/core-services/*.sh` | initialisation unique : montages, sysctl, etc. |
| 2 | `/etc/runit/2` | lance `runsvdir`, qui supervise tous les services activés |
| 3 | `/etc/runit/3` | arrêt du système |

Les acteurs à retenir :

- **`runsvdir`** surveille un répertoire et lance un superviseur **`runsv`** pour chaque service qu'il y trouve.
- **`runsv`** démarre le programme du service, le **relance automatiquement** s'il s'arrête, et accepte des ordres via la commande **`sv`**.
- Un service est donc un **dossier** contenant un script `run`.

Deux emplacements à distinguer :

| Dossier | Contenu |
|---|---|
| `/etc/sv/` | **définitions** des services disponibles (un dossier par service) |
| `/var/service/` | services **activés** : des liens symboliques vers `/etc/sv/<nom>` |

Activer un service = créer un lien symbolique vers sa définition. Le désactiver = supprimer le lien.

## 3.2 Comment ton `ln -s` active dhcpcd

Dans l'installation (partie 1), tu as tapé :

```sh
ln -s /etc/sv/dhcpcd /etc/runit/runsvdir/default/dhcpcd
```

Voici ce que cela fait :

- `/etc/sv/dhcpcd` est la **définition** du service (installée par le paquet `dhcpcd`, avec son script `run`).
- `/etc/runit/runsvdir/default/` est le dossier de services du « niveau d'exécution » **default**, c'est-à-dire celui que `runsvdir` charge au démarrage.
- Le lien symbolique fait apparaître le service dans ce dossier : au prochain boot, `runsvdir` le voit, lance un `runsv`, qui exécute `run`.

Pourquoi ce chemin dans le chroot, plutôt que `/var/service` ? Dans un chroot, runit n'est **pas en cours d'exécution** : on prépare simplement l'état au prochain démarrage, en ciblant directement le dossier `default`. Sur un système démarré, on utilise `/var/service`, qui pointe vers le dossier de services actif (vérifie avec `ls -ld /var/service /etc/runit/runsvdir/*`). Cela donne alors :

```sh
ln -s /etc/sv/dhcpcd /var/service/
```

Autre conséquence : dans le chroot, la commande `sv` ne marche pas (aucun `runsv` à contacter). Elle ne devient utile qu'une fois le système démarré.

## 3.3 Gérer les services avec `sv`

Les commandes s'exécutent en root et prennent le nom du service (ou son chemin).

| Action | Commande |
|---|---|
| **Activer** au démarrage (et le lancer tout de suite) | `ln -s /etc/sv/<nom> /var/service/` |
| **Désactiver** et l'arrêter | `rm /var/service/<nom>` |
| Voir l'état | `sv status <nom>` |
| Voir tous les états | `sv status /var/service/*` |
| Démarrer | `sv up <nom>` (ou `sv start`) |
| Arrêter (sans désactiver) | `sv down <nom>` (ou `sv stop`) |
| Redémarrer | `sv restart <nom>` |
| Lancer une fois, sans relance auto | `sv once <nom>` |
| Recharger la config (signal HUP) | `sv hup <nom>` (ou `sv reload`) |
| Tester si le service est sain | `sv check <nom>` |

Points à retenir :

- `sv down` arrête le service **jusqu'au prochain démarrage** de la machine ; il reste enregistré. Pour le désactiver pour de bon, il faut supprimer le lien.
- Si tu supprimes le lien dans `/var/service/`, `runsvdir` s'en aperçoit au bout de quelques secondes et arrête le service.
- Pour qu'un service soit enregistré **sans démarrer** automatiquement, créer un fichier `down` dans sa définition : `touch /etc/sv/<nom>/down`. Il pourra être lancé à la main avec `sv up <nom>`.

Exemples tirés du lab :

```sh
ln -s /etc/sv/dnsmasq /var/service/
sv status dnsmasq
sv restart dnsmasq
rm /var/service/dnsmasq
```

## 3.4 Créer un service from scratch

Prenons un exemple : un petit serveur web de test sur le port 8080.

### Étape 1 : créer le dossier du service

```sh
mkdir -p /etc/sv/monservice
```

### Étape 2 : écrire le script `run`

```sh
vim /etc/sv/monservice/run
```

```sh
#!/bin/sh
exec 2>&1
exec python3 -m http.server 8080 --directory /srv/www
```

Règles du script `run` :

- Il doit s'exécuter **au premier plan** : le programme ne doit **pas** se mettre en arrière-plan (« daemonize ») tout seul. Si le programme a une option du type `--foreground`, `--no-daemon` ou `--keep-in-foreground` (par exemple `dnsmasq --keep-in-foreground`), l'utiliser.
- `exec` remplace le script par le programme : le superviseur suit alors directement le bon processus et peut l'arrêter proprement.
- `exec 2>&1` redirige les erreurs vers la sortie standard, pour que les deux atterrissent au même endroit (le logger).
- Le fichier doit être **exécutable**.

### Étape 3 : rendre exécutable

```sh
chmod +x /etc/sv/monservice/run
```

### Étape 4 : activer

```sh
ln -s /etc/sv/monservice /var/service/
sv status monservice
```

Dans les secondes qui suivent, `runsvdir` détecte le lien et lance le service. S'il plante, `runsv` le relance automatiquement (avec une seconde d'attente).

### Fichiers optionnels du dossier de service

| Fichier | Rôle |
|---|---|
| `run` | **obligatoire** : démarre le programme |
| `finish` | exécuté quand le programme s'arrête (nettoyage) |
| `check` | script appelé par `sv check` pour tester la santé du service |
| `conf` | variables d'environnement, chargées par le script `run` (convention Void) |
| `down` | si présent, le service ne démarre pas automatiquement |
| `log/run` | lance un logger qui reçoit la sortie du service |

Exemple de `run` qui lit un fichier `conf` :

```sh
#!/bin/sh
[ -r ./conf ] && . ./conf
exec 2>&1
exec /usr/bin/monprogramme $OPTS
```

### Ajouter des logs avec `svlogd`

```sh
mkdir -p /etc/sv/monservice/log /var/log/monservice
```

`/etc/sv/monservice/log/run` :

```sh
#!/bin/sh
exec svlogd -tt /var/log/monservice
```

```sh
chmod +x /etc/sv/monservice/log/run
```

`svlogd` écrit la sortie du service (avec horodatage) dans `/var/log/monservice/current` et fait tourner les fichiers automatiquement.

### Dossier `supervise` (convention Void)

Les services officiels de Void ont un lien `supervise` pointant vers `/run/runit/supervise.<nom>`, ce qui garde les fichiers d'état de la supervision en mémoire (tmpfs) plutôt que sur le disque. Ce lien est celui que l'on voit dans l'exemple du handbook sur dhcpcd (`ln -nsf /run/runit/supervise.dhcpcd-enp3s0 ...`). Pour un service maison, c'est facultatif : si le lien n'existe pas, `runsv` crée un dossier `supervise` normal dans `/etc/sv/monservice/`.

```sh
ln -nsf /run/runit/supervise.monservice /etc/sv/monservice/supervise
```

## 3.5 Cas pratique : dupliquer un service existant

C'est ce que fait le handbook pour lancer dhcpcd sur **une seule interface**. On copie la définition, on adapte le script, on active :

```sh
cp -R /etc/sv/dhcpcd-eth0 /etc/sv/dhcpcd-enp0s3
ln -nsf /run/runit/supervise.dhcpcd-enp0s3 /etc/sv/dhcpcd-enp0s3/supervise
sed -i 's/eth0/enp0s3/' /etc/sv/dhcpcd-enp0s3/run
ln -s /etc/sv/dhcpcd-enp0s3 /var/service/
```

Étape par étape :

1. `cp -R` : copie de la définition existante sous un nouveau nom.
2. `ln -nsf` : le dossier `supervise` pointe vers `/run/...` (le `-n` évite de suivre un lien existant, le `-f` remplace).
3. `sed -i` : remplace `eth0` par le vrai nom de l'interface dans le script `run`.
4. `ln -s` : active le nouveau service.

## 3.6 Dépendances entre services

runit n'a **pas de système de dépendances** (contrairement à systemd). Un service qui a besoin d'un autre le gère lui-même dans `run`, avec `sv check`. Exemple : un service qui doit attendre que dhcpcd tourne.

```sh
#!/bin/sh
sv check dhcpcd >/dev/null || exit 1
exec 2>&1
exec monprogramme
```

Si la dépendance n'est pas prête, le script se termine, `runsv` le relance une seconde plus tard, et ainsi de suite jusqu'à ce que la dépendance soit satisfaite.

## 3.7 Journaux du système

Void n'installe aucun démon de journalisation par défaut : les messages du noyau et des services « disparaissent ». La solution recommandée, qui s'intègre à runit, est **socklog** :

```sh
xbps-install socklog-void
ln -s /etc/sv/socklog-unix /var/service/
ln -s /etc/sv/nanoklogd /var/service/
```

- `socklog-unix` récupère les messages `syslog` des programmes ;
- `nanoklogd` récupère ceux du noyau ;
- les journaux sont écrits sous `/var/log/socklog/`.

En cas de problème avec un service, un test direct est souvent le plus rapide : arrêter le service (`sv down <nom>`) puis lancer sa commande à la main au premier plan pour voir les erreurs.

---

## Sources

- Post-installation de Void (socklog, nftables) : <https://computingforgeeks.com/void-linux-post-install-setup/>
