---
parent: Void Linux
title: 02 Projet Voidrouter
layout: default
---

# Partie 2 : Le lab routeur

## 2.1 Objectif et matériel

Construire un petit **routeur / firewall** minimal :

- un port **WAN** en DHCP (vers la box ou Internet),
- un port **LAN** en IP fixe, avec un serveur DHCP/DNS pour les clients,
- du routage avec NAT et un firewall.

Matériel cible : un **Raspberry Pi Zero 2W** avec la carte d'extension **RPi-Zero-Dual-ETH**, qui fournit deux ports Ethernet 100 Mbps. Le guide du fabricant montre que `lsusb` voit deux cartes réseau distinctes (donc deux interfaces indépendantes dans l'OS), avec environ 93 Mbps par port en test.

Limite à connaître : le Zero 2W n'a qu'un seul bus USB 2.0, que les deux ports se partagent. Le gigabit n'aurait aucun intérêt ici.

Détails de la carte, d'après le guide du fabricant :

- Elle se fixe sur le Zero par des pastilles (alimentation et données USB) et fonctionne comme un **hub USB à 4 ports** : 2 pour les adaptateurs Ethernet 100 Mbps et **2 ports USB 2.0 de type A** (fusibles de 2 A). Un de ces ports peut servir à un SSD ou à une clé USB.
- Le port micro-USB OTG du Pi ne doit **rien** recevoir, sinon l'USB de la carte ne fonctionne plus.
- Alimentation : soit par le micro-USB marqué PWR du Pi, soit par le USB-C de la carte, **jamais les deux en même temps**.
- Dans `config.txt` (sur le support de démarrage) : `dtoverlay=dwc2,dr_mode=host` (aux deux endroits où il apparaît dans le fichier), et `otg_mode=1` de préférence commenté. Sans cela, l'USB peut ne pas fonctionner.
- Les deux adaptateurs apparaissent en `eth0`/`eth1` (Raspberry Pi OS) ou `enx<MAC>` (Ubuntu). Une version récente de la carte réserve un connecteur pour un module 4G (voir 2.17).

### Environnement de test : une VM Void musl

Pour tester sans le Pi, la VM reçoit **deux cartes réseau** :

| Carte | Rôle | Mode VMware | Interface | Adressage |
|---|---|---|---|---|
| 1 | WAN | NAT (VMware fournit un DHCP et un accès Internet) | `eno16777736` | DHCP (dhcpcd) |
| 2 | LAN | réseau isolé, sans DHCP | `eno33554960` | `192.168.10.1/24` (fixe) |

Sous VMware, les interfaces s'appellent `eno...` (voir 2.3). Dans le reste du guide, certains exemples génériques utilisent encore `enp0s3` et `enp0s8` (noms typiques d'une autre configuration) : remplace-les par les noms ci-dessus, que tu peux vérifier avec `ip -br link`.

Une seconde VM cliente, branchée sur le même réseau interne, sert à vérifier que le DHCP, le DNS et le routage fonctionnent.

## 2.2 Architecture

```
Internet / box
      │
 [port WAN]   dhcpcd (client DHCP)
      │
┌─────┴──────────────┐
│  Void Linux        │   routage (sysctl) + firewall/NAT (nftables)
│  Pi Zero 2W / VM   │
└─────┬──────────────┘
 [port LAN]   192.168.10.1/24 (IP fixe)   dnsmasq (DHCP + DNS)
      │
 switch / clients
```

Pile logicielle, de bas en haut :

1. **Noyau + udev** : créent et nomment les interfaces.
2. **`ip` + runit** : configuration statique et démarrage des services.
3. **dhcpcd** (WAN) : obtient IP, passerelle et DNS.
4. **dnsmasq** (LAN) : serveur DHCP et relais DNS pour les clients.
5. **sysctl** : active le routage entre les interfaces.
6. **nftables** : filtrage et NAT.
7. **wpa_supplicant** (optionnel) : seulement pour le Wi-Fi.

Chemin d'un paquet : un client du LAN obtient son IP via **dnsmasq**, envoie son trafic vers `192.168.10.1`, le noyau le **route** (sysctl), **nftables** applique les règles et le NAT, puis le paquet repart par le WAN avec l'adresse fournie par **dhcpcd**.

## 2.3 Noyau et udev : les interfaces

Le **noyau** crée les interfaces grâce aux drivers (par exemple `asix` ou `r8152` pour de l'Ethernet USB, `brcmfmac` pour le Wi-Fi du Pi). **udev** leur donne ensuite un nom.

- Par défaut, udev utilise des noms **prédictibles** : `enp0s3` (PCI), `eno<N>` (index fourni par le firmware, ce qui est le cas sous VMware : `eno16777736` et `eno33554960`), `enx<MAC>` (USB), `wlan0` ou `wlp...` (Wi-Fi).
- Pour revenir à `eth0`, `eth1`, ajouter `net.ifnames=0` à la ligne de commande du noyau (GRUB : `GRUB_CMDLINE_LINUX_DEFAULT` dans `/etc/default/grub`, puis `grub-mkconfig`). Attention : avec des interfaces USB, l'ordre `eth0`/`eth1` peut changer d'un démarrage à l'autre.
- Pour **figer un nom par adresse MAC**, créer une règle dans `/etc/udev/rules.d/10-network.rules` :

```
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="aa:bb:cc:dd:ee:ff", NAME="wan0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="aa:bb:cc:dd:ee:00", NAME="lan0"
```

Commandes pour inspecter :

```sh
ip -br link
ip -br addr
lsusb
```

## 2.4 `ip` (iproute2) : l'outil de base

Il configure tout au niveau noyau : état des liens, adresses, routes, bridges, VLAN. Les autres outils (dhcpcd, NetworkManager...) font la même chose en arrière-plan.

La commande est organisée en **objets** : `ip link` (interfaces), `ip addr` (adresses) et `ip route` (routes). Chaque objet accepte `show`, `add`, `del`, `set`... L'option `-br` (brief) donne une sortie compacte, et `-s` ajoute des statistiques.

### Gérer les interfaces (`ip link`)

```sh
ip -br link                                        # liste compacte des interfaces et de leur état
ip link show dev enp0s3                            # détail d'une interface
ip -s link show dev enp0s3                         # statistiques (paquets, erreurs, pertes)

ip link set dev enp0s3 up                          # activer
ip link set dev enp0s3 down                        # désactiver

ip link set dev enp0s3 mtu 1400                    # changer le MTU
ip link set dev enp0s3 address aa:bb:cc:dd:ee:ff  # changer l'adresse MAC (interface down)
ip link set dev enp0s3 name wan0                   # renommer (interface down)
```

États à connaître dans la sortie :

| État | Signification |
|---|---|
| `UP` | interface activée et lien présent |
| `DOWN` | interface désactivée : `ip link set dev ... up` |
| `NO-CARRIER` | interface activée, mais **aucun lien physique** (câble débranché, câble virtuel déconnecté dans la VM) |

### Gérer les adresses (`ip addr`)

```sh
ip -br addr                                        # adresses de toutes les interfaces
ip addr add 192.168.10.1/24 brd + dev enp0s8       # ajouter une adresse
ip addr del 192.168.10.1/24 dev enp0s8             # supprimer une adresse précise
ip addr flush dev enp0s8                           # supprimer toutes les adresses de l'interface
```

- `ip addr add` **ajoute** une adresse sans remplacer les autres : une interface peut en avoir plusieurs.
- `brd +` calcule automatiquement l'adresse de broadcast.
- Monter une interface ne lui donne **pas** d'IP : elle vient de dhcpcd ou d'un `ip addr add`.

### Le routage (`ip route`)

La **table de routage** indique au noyau par quelle interface et via quelle passerelle envoyer chaque paquet selon son adresse de destination. Le noyau choisit la route la plus **précise** (le préfixe le plus long) ; si aucune ne correspond, il utilise la route **par défaut**.

```sh
ip route                                           # affiche la table de routage (IPv4)
ip route show default                              # uniquement la route par défaut
ip route get 1.1.1.1                               # montre la route que le noyau utiliserait pour cette destination

ip route add default via 192.168.1.1 dev enp0s3    # route par défaut via une passerelle
ip route add 10.20.0.0/16 via 192.168.10.254       # réseau distant, atteint via un routeur voisin
ip route add 10.30.0.0/24 dev enp0s8               # réseau directement relié à cette interface
ip route replace default via 192.168.1.1           # remplace la route par défaut (ou la crée)
ip route del 10.20.0.0/16                          # supprime une route
```

Exemple de sortie sur le routeur du lab (simplifié, l'affichage exact varie) :

```
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3 scope link
192.168.10.0/24 dev enp0s8 scope link
```

- La première ligne est la **route par défaut** : tout ce qui n'est pas connu part vers la box, côté WAN.
- Les deux suivantes sont des **routes de réseau directement relié** : le noyau les crée tout seul quand une interface reçoit une adresse.

Points à connaître :

- **`via`** désigne la passerelle (le « prochain saut »). Elle doit se trouver sur un réseau déjà directement joignable. Sinon, l'erreur est `Network is unreachable` : il faut d'abord donner l'adresse à l'interface, puis ajouter la route.
- **`metric`** : quand plusieurs routes couvrent la même destination, celle qui a la métrique la plus basse est utilisée (par exemple `ip route add default via 192.168.1.1 metric 100`).
- **Sur ton routeur**, c'est dhcpcd qui installe la route par défaut côté WAN : évite d'en ajouter une autre à la main sur la même interface. Le LAN n'a besoin d'aucune route supplémentaire tant que tous les clients sont dans `192.168.10.0/24`.
- **Router vraiment** (faire passer des paquets d'une interface à l'autre) demande en plus d'activer le forwarding (voir 2.8).
- Comme le reste, les routes ajoutées avec `ip` sont **perdues au redémarrage**.

Ces réglages ne survivent pas au redémarrage. Pour les rendre permanents, deux options :

- les mettre dans **`/etc/rc.local`** (script exécuté à la fin du démarrage) ;
- ou les déclarer dans `/etc/dhcpcd.conf` (bloc `static`, voir plus bas).

Exemple de `/etc/rc.local` pour le LAN du lab :

```sh
ip link set dev eno33554960 up
ip addr add 192.168.10.1/24 brd + dev eno33554960
```

## 2.5 dhcpcd : le client DHCP

**Rôle.** dhcpcd *demande* une adresse IP, une passerelle et des DNS à un serveur DHCP, puis configure l'interface. Il écrit aussi `/etc/resolv.conf` à partir des DNS reçus. Il n'est **pas** un serveur DNS ni un serveur DHCP.

Sur Void, un service `dhcpcd` existe. Selon la doc, il est actif d'office avec une installation faite par le programme d'installation. Avec une installation manuelle par `xbps-install -r` (comme dans la partie 1), il faut l'activer à la main.

Par défaut, il tourne sur **toutes** les interfaces, ce qu'on ne veut pas sur un routeur (il tenterait de demander une IP sur le LAN, où c'est dnsmasq le serveur).

### Option A : un service global, en excluant le LAN (celle utilisée dans le lab)

Dans `/etc/dhcpcd.conf` :

```
denyinterfaces eno33554960
```

Syntaxe de `denyinterfaces` :

- c'est une directive **globale**, sur une seule ligne : `denyinterfaces pattern` ;
- le `pattern` est une **liste de motifs séparés par des espaces ou des virgules**, passés à `fnmatch(3)`, donc les jokers fonctionnent : `denyinterfaces eno33554960 enx*` ou `denyinterfaces eno33554960,wlan0` (les deux écritures sont acceptées, évite les espaces après les virgules) ;
- `denyinterfaces` l'emporte sur `allowinterfaces` ;
- à placer **avant** tout bloc `interface ...` : après une ligne `interface <nom>`, les options suivantes ne s'appliquent qu'à cette interface.

Variante pour un script : lancer dhcpcd en lui passant l'interface en argument, ce qui limite sa gestion à celle-ci (`dhcpcd -B eno16777736`, où `-B` laisse dhcpcd au premier plan).

### Option B : un service par interface (méthode du handbook Void)

```sh
sv down dhcpcd
rm /var/service/dhcpcd
cp -R /etc/sv/dhcpcd-eth0 /etc/sv/dhcpcd-enp0s3
ln -nsf /run/runit/supervise.dhcpcd-enp0s3 /etc/sv/dhcpcd-enp0s3/supervise
sed -i 's/eth0/enp0s3/' /etc/sv/dhcpcd-enp0s3/run
ln -s /etc/sv/dhcpcd-enp0s3 /var/service/
```

### Quand aucun serveur DHCP ne répond

- dhcpcd attend la réponse pendant **30 secondes** par défaut (option `timeout`, `0` = attendre sans limite).
- Sans réponse, il se donne une adresse **link-local** `169.254.x.x` (IPv4LL, RFC 3927) puis continue de chercher un serveur DHCP.
- Ces adresses ne sont **jamais routées** : elles ne sortent pas du lien local. Voir une IP en `169.254` sur une machine est le signe classique d'un DHCP qui n'a pas répondu.

Options utiles dans `/etc/dhcpcd.conf` :

```
# Désactive l'adresse 169.254 en cas d'échec
noipv4ll

# IP de secours statique si le DHCP échoue
profile secours
static ip_address=10.0.2.50/24
static routers=10.0.2.2

interface enp0s3
fallback secours

# IP statique permanente sur une interface
interface enp0s8
static ip_address=192.168.10.1/24

# Ne pas toucher à /etc/resolv.conf (si un DNS local gère le fichier)
nohook resolv.conf
```

### Commandes utiles

```sh
sv status dhcpcd
sv restart dhcpcd
dhcpcd -d -B enp0s3       # lancement manuel, debug, au premier plan
dhcpcd -n enp0s3          # force un renouvellement du bail
```

Les pages de manuel consultées pour ces comportements sont celles de versions anciennes de dhcpcd. Pour la version de ta machine : `man dhcpcd` et `man dhcpcd.conf`.

## 2.6 wpa_supplicant : le Wi-Fi (optionnel)

**Rôle.** Il gère uniquement l'**authentification** Wi-Fi (WPA/WPA2). Il ne donne pas d'adresse IP : dhcpcd (ou une IP statique) intervient ensuite sur l'interface Wi-Fi. Alternative plus récente : `iwd`.

Configuration, dans `/etc/wpa_supplicant/wpa_supplicant.conf`. On peut la générer avec :

```sh
wpa_passphrase "MonSSID" "MotDePasse" >> /etc/wpa_supplicant/wpa_supplicant.conf
```

Puis activer le service :

```sh
xbps-install wpa_supplicant
ln -s /etc/sv/wpa_supplicant /var/service/
```

Vérifie le contenu de `/etc/sv/wpa_supplicant/run` pour savoir quelle interface et quel fichier de conf le service utilise.

## 2.7 dnsmasq : serveur DHCP et DNS du LAN

**Rôle.** Petit démon qui fait à la fois :

- le **serveur DHCP** : distribue IP, passerelle et DNS aux clients du LAN (avec réservation par adresse MAC possible) ;
- un **relais DNS avec cache** : relaie les requêtes vers un DNS amont (`1.1.1.1`...) et garde les réponses en mémoire ;
- un **DNS local** : il répond aux noms de `/etc/hosts` et aux noms des machines qui ont un bail DHCP.

Il ne fait **pas** de résolution récursive complète : pour cela, il faut `unbound`.

**Complémentarité avec dhcpcd** : dhcpcd est un *client* (côté WAN), dnsmasq est un *serveur* (côté LAN).

Installation et activation :

```sh
xbps-install dnsmasq
ln -s /etc/sv/dnsmasq /var/service/
```

Configuration minimale, dans `/etc/dnsmasq.conf` :

```
interface=eno33554960
bind-interfaces
dhcp-range=192.168.10.50,192.168.10.150,12h
dhcp-option=option:router,192.168.10.1
dhcp-authoritative
server=1.1.1.1
no-resolv
domain-needed
bogus-priv
log-dhcp
```

- `interface` et `bind-interfaces` : écoute uniquement sur le LAN.
- `dhcp-range` : plage distribuée et durée du bail.
- `dhcp-authoritative` : dnsmasq se considère comme le seul serveur DHCP du réseau local.
- `server` et `no-resolv` : DNS amont explicite. `no-resolv` dit à dnsmasq de **ne pas lire `/etc/resolv.conf`** : sans cela, un `resolv.conf` qui pointe vers la machine elle-même créerait une boucle, et le DNS amont changerait avec ce que la box annonce à dhcpcd.
- `domain-needed` et `bogus-priv` : évitent de faire fuir des requêtes locales vers l'extérieur.
- `log-dhcp` : logs détaillés du DHCP, utile pendant la mise au point (à retirer ensuite).

Lancement et vérification (utile pour un script ou un service runit) :

```sh
dnsmasq --test -C /etc/dnsmasq.conf                    # vérifie la syntaxe, ne démarre rien
dnsmasq -k -C /etc/dnsmasq.conf --log-facility=-       # premier plan, logs sur stderr
```

- `--test` quitte avec le code 0 si la configuration est correcte.
- `-k` (`--keep-in-foreground`) est prévu pour les superviseurs de type daemontools, donc runit. À ne pas confondre avec `-d`, réservé au débogage.
- `--log-facility=-` envoie les logs sur stderr. Les baux sont stockés par défaut dans `/var/lib/misc/dnsmasq.leases` (option `-l` pour changer).
- Options voisines : `resolv-file=<chemin>` (lire un autre fichier que `/etc/resolv.conf`) et `no-poll` (ne pas surveiller les changements de `resolv.conf`).

### Qui possède `/etc/resolv.conf` ?

Plusieurs outils veulent écrire ce fichier :

- dhcpcd l'écrit à partir du DHCP du WAN (désactivable avec `nohook resolv.conf`) ;
- dnsmasq ne l'écrit pas, il le **lit** (sauf `no-resolv`).

Il faut décider qui en est responsable pour éviter qu'ils s'écrasent entre eux.

## 2.8 Forwarding IPv4 (sysctl)

Par défaut, le noyau ne transmet pas les paquets d'une interface à l'autre. Pour faire un routeur :

```sh
sysctl -w net.ipv4.ip_forward=1        # immédiat
```

Pour que ce soit permanent, créer `/etc/sysctl.d/10-forward.conf` :

```
net.ipv4.ip_forward=1
```

Vérification :

```sh
sysctl net.ipv4.ip_forward
```

## 2.9 nftables : firewall et NAT

**Rôle.** **netfilter** est le firewall *dans le noyau*. **nftables** est l'outil en espace utilisateur (commande `nft`) qui définit ses règles. Il remplace iptables. Void fournit `iptables` dans le système de base ; nftables est à installer.

```sh
xbps-install nftables runit-nftables
```

Le paquet `runit-nftables` recharge le ruleset au démarrage depuis **`/etc/nftables.conf`**.

> Attention au chemin : le service Void lit `/etc/nftables.conf`, pas `/etc/nftables/nftables.conf` comme sur beaucoup d'autres distributions.

### Configuration utilisée dans le lab (NAT depuis le LAN uniquement)

Version minimale, qui a été validée dans le lab : `/etc/nftables.conf`.

```
#!/usr/sbin/nft -f

flush ruleset

define WAN = "eno16777736"
define LAN = "eno33554960"

# Filtering of traffic going THROUGH the router: only the LAN can initiate toward the WAN
table inet filter {
    chain forward {
        type filter hook forward priority 0; policy drop;

        # Allow replies to connections already initiated from the LAN
        ct state established,related accept
        ct state invalid drop

        # Allow new connections from the LAN to the WAN only
        iifname $LAN oifname $WAN accept
    }
}

# NAT overload (masquerade): applied only to traffic coming from the LAN
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;

        # Rewrite the source address with the WAN interface address
        iifname $LAN oifname $WAN masquerade
    }
}
```

- Seuls les paquets entrés par le LAN et sortant par le WAN sont traduits (NAT overload). Avec un sous-réseau fixe, on peut aussi filtrer par adresse : `ip saddr 192.168.10.0/24 oifname $WAN masquerade`.
- La chaîne `forward` en `policy drop` autorise les connexions initiées depuis le LAN, les réponses (`established,related`), et rien depuis le WAN vers le LAN.
- Cette version ne contient **pas** de chaîne `input` : le routeur lui-même n'est pas filtré (politique `accept` par défaut).
- Le fichier est chargé au démarrage depuis `/etc/nftables.conf` (voir plus haut pour le chemin). On peut aussi le charger à la main avec `nft -f <fichier>` : le `flush ruleset` en tête rend le rechargement sans danger.

### Version durcie (optionnelle, non testée dans le lab)

La version ci-dessous ajoute une chaîne `input` (le routeur lui-même est fermé depuis le WAN) :

```
#!/usr/sbin/nft -f

flush ruleset

define WAN = "eno16777736"
define LAN = "eno33554960"

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        iif lo accept
        ct state established,related accept
        ct state invalid drop

        # DHCP replies on the WAN (otherwise dhcpcd never gets a lease)
        iifname $WAN udp sport 67 udp dport 68 accept

        # ICMP (ping) and ICMPv6 (required for IPv6 to work)
        ip protocol icmp accept
        ip6 nexthdr icmpv6 accept

        # Services offered to the LAN: DHCP, DNS, SSH
        iifname $LAN udp dport { 53, 67 } accept
        iifname $LAN tcp dport { 22, 53 } accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;

        ct state established,related accept
        iifname $LAN oifname $WAN accept
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}

table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname $WAN masquerade
    }
}
```

Points clés :

- `policy drop` en entrée : tout est refusé sauf ce qui est explicitement autorisé.
- La règle DHCP du WAN est nécessaire : sans elle, les réponses du serveur DHCP sont bloquées et dhcpcd n'obtient jamais d'adresse.
- `masquerade` réalise le NAT : les clients du LAN sortent avec l'IP du WAN.
- Ne pas bloquer ICMPv6 en bloc : il est indispensable en IPv6 (découverte des voisins, annonces de routeur, découverte du MTU).

Vérification et activation :

```sh
nft -c -f /etc/nftables.conf        # vérifie la syntaxe sans appliquer
ln -s /etc/sv/nftables /var/service/ # si le service n'est pas déjà actif
nft list ruleset                    # affiche les règles chargées
```

## 2.10 Notions utiles : IPv4LL, privé et IPv6

| Type d'adresse | Plage | Routable ? |
|---|---|---|
| Publique | le reste | Oui, sur Internet |
| Privée (RFC 1918) | `10/8`, `172.16/12`, `192.168/16` | En interne, pas sur Internet (NAT nécessaire) |
| Link-local (IPv4LL) | `169.254.0.0/16` | **Non**, jamais routée, valable sur le lien local seulement |

- La plage `169.254/16` n'est pas privée : c'est une plage **à usage spécial** réservée par l'IANA, que personne n'annonce sur Internet.
- Équivalent IPv6 du link-local : `fe80::/10`, présent en permanence sur chaque interface.

**NAT en IPv6** : en pratique, on n'en fait pas. Le FAI délègue un **préfixe** entier (DHCPv6 Prefix Delegation), chaque machine reçoit une adresse globale (SLAAC). La protection qu'offrait le NAT en IPv4 vient alors d'un **pare-feu à états** (autoriser le sortant et ses réponses, bloquer le reste en entrée). Le NAT66 existe sous Linux mais est déconseillé ; NPTv6 (traduction de préfixe sans état) et NAT64 répondent à des cas précis.

## 2.11 Tableau récapitulatif des fichiers

| Fichier | Rôle |
|---|---|
| `/etc/udev/rules.d/*.rules` | nommer les interfaces (par MAC) |
| `/etc/default/grub` | options du noyau (`net.ifnames=0`) |
| `/etc/rc.local` | commandes `ip` exécutées au démarrage (IP statique du LAN) |
| `/etc/dhcpcd.conf` | client DHCP du WAN : `denyinterfaces eno33554960`, fallback |
| `/etc/sv/dhcpcd*` | service(s) runit de dhcpcd |
| `/etc/wpa_supplicant/wpa_supplicant.conf` | réseaux Wi-Fi |
| `/etc/dnsmasq.conf` | DHCP et DNS du LAN |
| `/etc/resolv.conf` | DNS utilisés par la machine elle-même |
| `/etc/sysctl.d/*.conf` | forwarding IPv4 |
| `/etc/nftables.conf` | firewall et NAT |

## 2.12 Tests de bout en bout

Sur le routeur :

```sh
ip -br addr                  # WAN en DHCP, LAN en 192.168.10.1
ip route                     # route par défaut via le WAN
sysctl net.ipv4.ip_forward   # = 1
nft list ruleset             # règles chargées
sv status dhcpcd dnsmasq nftables
```

Sur un client du LAN (seconde VM) :

```sh
ip -br addr                  # IP dans 192.168.10.50-150
ip route                     # passerelle 192.168.10.1
ping 192.168.10.1            # le routeur répond
ping 1.1.1.1                 # le routage et le NAT fonctionnent
ping example.com             # le DNS fonctionne
```

En cas de souci :

- Pas d'IP sur le WAN : `dhcpcd -d -B eno16777736` pour voir les échanges ; si tu utilises la version durcie de nftables, vérifier la règle DHCP du WAN.
- Le client ne reçoit pas d'IP : vérifier `interface=` dans dnsmasq et que dhcpcd est bien exclu du LAN.
- Ping IP qui marche, mais pas les noms : problème DNS (voir `server=` et `/etc/resolv.conf`).
- Aucun ping vers l'extérieur : vérifier `ip_forward`, la règle `forward` et le `masquerade`.

## 2.13 Configuration finale de la VM (VMware)

Récapitulatif de la configuration validée dans le lab, dans l'ordre. L'adressage (`192.168.10.0/24`, plage de `.50` à `.150`) est celui des exemples de ce guide : adapte-le si tu en as choisi un autre. WAN : `eno16777736` (DHCP). LAN : `eno33554960` (IP fixe).

### Étape 1 : paquets

```sh
xbps-install dhcpcd dnsmasq nftables runit-nftables
```

### Étape 2 : dhcpcd ne doit pas toucher au LAN (`/etc/dhcpcd.conf`)

```
denyinterfaces eno33554960
```

### Étape 3 : IP du LAN (`/etc/rc.local`, ou les mêmes commandes dans ton orchestrateur)

```sh
ip link set dev eno33554960 up
ip addr add 192.168.10.1/24 brd + dev eno33554960
```

### Étape 4 : DHCP et DNS du LAN (`/etc/dnsmasq.conf`)

```
interface=eno33554960
bind-interfaces
dhcp-range=192.168.10.50,192.168.10.150,12h
dhcp-option=option:router,192.168.10.1
dhcp-authoritative
server=1.1.1.1
no-resolv
domain-needed
bogus-priv
log-dhcp
```

### Étape 5 : forwarding (`/etc/sysctl.d/10-forward.conf`)

```
net.ipv4.ip_forward=1
```

Pour l'appliquer tout de suite : `sysctl -w net.ipv4.ip_forward=1`.

### Étape 6 : firewall et NAT (`/etc/nftables.conf`)

```
#!/usr/sbin/nft -f

flush ruleset

define WAN = "eno16777736"
define LAN = "eno33554960"

# Filtering of traffic going THROUGH the router: only the LAN can initiate toward the WAN
table inet filter {
    chain forward {
        type filter hook forward priority 0; policy drop;

        # Allow replies to connections already initiated from the LAN
        ct state established,related accept
        ct state invalid drop

        # Allow new connections from the LAN to the WAN only
        iifname $LAN oifname $WAN accept
    }
}

# NAT overload (masquerade): applied only to traffic coming from the LAN
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;

        # Rewrite the source address with the WAN interface address
        iifname $LAN oifname $WAN masquerade
    }
}
```

### Étape 7 : activer les services

```sh
ln -s /etc/sv/dhcpcd /var/service/
ln -s /etc/sv/dnsmasq /var/service/
```

Selon la doc Void, `runit-nftables` ajoute un *core service* qui recharge `/etc/nftables.conf` au démarrage. Certains guides activent en plus `ln -s /etc/sv/nftables /var/service/`. Après un redémarrage, vérifie avec `nft list ruleset` que les règles sont bien chargées.

### Étape 8 : vérifications

```sh
nft -c -f /etc/nftables.conf                  # syntaxe du firewall
dnsmasq --test -C /etc/dnsmasq.conf           # syntaxe de dnsmasq
ip -br addr                                   # WAN en DHCP, LAN en 192.168.10.1/24
ip route                                      # route par défaut via le WAN
sysctl net.ipv4.ip_forward                    # = 1
sv status dhcpcd dnsmasq
```

Puis, depuis un client du LAN : IP reçue dans la plage, `ping 192.168.10.1`, `ping 1.1.1.1`, `ping example.com`.

## 2.14 Ordre de démarrage (pour l'orchestrateur)

Ordre logique, avec le raisonnement derrière chaque étape :

1. **IP du LAN** (`ip link set ... up`, `ip addr add ...`) : l'interface doit exister avec son adresse avant que dnsmasq ne s'y lie (`bind-interfaces`).
2. **Forwarding** (`sysctl -w net.ipv4.ip_forward=1`).
3. **Firewall et NAT** : `nft -c -f /etc/nftables.conf && nft -f /etc/nftables.conf`.
4. **dhcpcd sur le WAN** : `dhcpcd -B eno16777736` (reste au premier plan). Avec `denyinterfaces eno33554960` dans `dhcpcd.conf`, ou en passant uniquement l'interface WAN en argument, le LAN n'est pas touché.
5. **dnsmasq** en dernier : `dnsmasq --test -C /etc/dnsmasq.conf && dnsmasq -k -C /etc/dnsmasq.conf --log-facility=-`.

dhcpcd (avec `-B`) et dnsmasq (avec `-k`) restent au premier plan : un script qui les lance doit les mettre en arrière-plan lui-même, ou les confier à runit (voir la partie 3).

Rappels bash utiles pour l'orchestrateur :

```bash
set -euo pipefail

: "${WAN:=eno16777736}"                        # valeur par défaut si WAN est vide ou absent
if [ -z "${1:-}" ]; then echo "usage: $0 <arg>" >&2; exit 1; fi   # argument absent ?

wan_ip=$(ip -4 -o addr show dev "$WAN" | awk '{print $4}')        # capturer une sortie
local_var=""                                   # dans une fonction : déclarer d'abord
# local x; x=$(commande)                       # et affecter ensuite, pour ne pas masquer le code de retour
```

## 2.15 Alternatives à dnsmasq pour le DHCP

| | dnsmasq | ISC DHCP (paquet Void `dhcp`) | Kea |
|---|---|---|---|
| Format de config | lignes `clé=valeur` | blocs `subnet { }` | JSON |
| DNS inclus | oui | non | non |
| Maintenance | active | terminée fin 2022 (dernière version 4.4.3-P1) | active (successeur d'ISC DHCP) |
| Poids | très léger | léger | plus lourd |

**ISC DHCP** : `isc-dhcp-server` est le nom Debian/Ubuntu ; sur Void, le paquet s'appelle `dhcp` (version 4.4.3P1, la dernière publiée). Il faut un autre démon pour le DNS. Exemple de `dhcpd.conf` pour le lab :

```
authoritative;
default-lease-time 43200;
max-lease-time 86400;

subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.50 192.168.10.150;
    option routers 192.168.10.1;
    option domain-name-servers 1.1.1.1;
}
```

Lancement, au premier plan et sur une interface précise (le fichier de baux `dhcpd.leases` doit exister avant) :

```sh
dhcpd -4 -f -cf /etc/dhcp/dhcpd.conf eno33554960
```

Les chemins sont ceux de Debian ; sous Void, regarde où le paquet installe ses fichiers avec `xbps-query -f dhcp`.

**Kea** : configuration JSON (exemple adapté de ceux d'ISC) :

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "eno33554960" ]
    },
    "lease-database": {
      "type": "memfile"
    },
    "valid-lifetime": 43200,
    "subnet4": [
      {
        "subnet": "192.168.10.0/24",
        "interface": "eno33554960",
        "pools": [ { "pool": "192.168.10.50 - 192.168.10.150" } ],
        "option-data": [
          { "name": "routers", "data": "192.168.10.1" },
          { "name": "domain-name-servers", "data": "1.1.1.1" }
        ]
      }
    ]
  },
  "Logging": {
    "loggers": [
      {
        "name": "kea-dhcp4",
        "output_options": [ { "output": "stdout" } ],
        "severity": "INFO"
      }
    ]
  }
}
```

```sh
kea-dhcp4 -t /etc/kea/kea-dhcp4.conf    # vérifie la syntaxe
kea-dhcp4 -c /etc/kea/kea-dhcp4.conf    # lance le serveur
```

Les exemples de Kea que j'ai trouvés viennent de versions anciennes : avec Kea 3.x (version dans les dépôts Void), certains noms de clés ont pu changer (par exemple `output_options` contre `output-options`). Le chemin `/etc/kea/` est une convention non vérifiée sur Void.

Pour le Zero 2W, dnsmasq reste le plus léger : un seul démon et un seul fichier pour le DHCP et le DNS.

## 2.16 VLAN sous Linux (802.1Q)

Linux gère les VLANs nativement avec le module `8021q` (`modprobe 8021q`).

### Sous-interfaces VLAN (côté « trunk »)

L'interface physique reçoit des trames **taguées** ; on crée une sous-interface par VLAN (routeur « sur un bras ») :

```sh
modprobe 8021q
ip link add link eno33554960 name eno33554960.10 type vlan id 10
ip link add link eno33554960 name eno33554960.20 type vlan id 20
ip link set eno33554960 up
ip link set eno33554960.10 up
ip link set eno33554960.20 up
ip addr add 192.168.10.1/24 dev eno33554960.10
ip addr add 192.168.20.1/24 dev eno33554960.20
```

Chaque sous-interface a ses propres IP, plage DHCP (dnsmasq) et règles nftables (`iifname "eno33554960.10"`).

### Bridge VLAN-aware (comportement de switch)

```sh
ip link add br0 type bridge vlan_filtering 1
ip link set <port_access> master br0
ip link set <port_trunk> master br0
ip link set br0 up

# Port access VLAN 10 : trames sans tag, mises dans le VLAN 10
bridge vlan del dev <port_access> vid 1
bridge vlan add dev <port_access> vid 10 pvid untagged

# Port trunk : VLANs 10 et 20 taguées
bridge vlan del dev <port_trunk> vid 1
bridge vlan add dev <port_trunk> vid 10
bridge vlan add dev <port_trunk> vid 20

bridge vlan show
```

| Concept | Commande `bridge vlan` |
|---|---|
| Port **access** VLAN X | `vid X pvid untagged` |
| Port **trunk** (tagué) | `vid X` (sans `untagged`) |
| **VLAN natif** sur un trunk | `vid X pvid untagged` sur le port trunk |

Pour donner une IP au routeur dans un VLAN d'un bridge : `bridge vlan add dev br0 vid 10 self`, puis `ip link add link br0 name br0.10 type vlan id 10`.

Limites : le bridge Linux commute en **logiciel**, sans accélération matérielle. Sur le Pi, les deux ports Ethernet partagent le bus USB 2.0. Teste avec un ping entre deux machines du même VLAN avant de faire confiance au montage, car certains adaptateurs USB peuvent mal gérer les trames taguées (la balise ajoute 4 octets). Sous un hyperviseur, vérifie aussi que le réseau virtuel laisse passer les tags.

## 2.17 Matériel : stockage, fiabilité de la carte SD et 4G

### Stockage sur le Pi Zero 2 W

- **Démarrage sur USB** : d'après une modification de la documentation officielle Raspberry Pi, le Zero 2 W accepte le démarrage sur stockage USB sans réglage préalable (des messages de 2021 évoquaient un bit OTP à activer avec `program_usb_boot_mode=1` ; ce bit est **définitif**). Il ne gère **pas** le démarrage réseau. Je n'ai pas vérifié si le boot ROM énumère un SSD derrière le hub de la carte Dual-ETH : à tester. Plan B : une petite SD avec seulement `/boot` (en lecture seule) et la racine sur le SSD.
- Un SSD est préférable à une clé USB bon marché, qui s'use autant qu'une SD et peut être plus lente. Tout reste sur un seul bus USB 2.0, partagé avec les deux Ethernet.
- **rpiboot** (dépôt `usbboot`) gère aussi le Zero 2 W, mais demande un ordinateur hôte branché en permanence : utile pour provisionner ou réparer, pas pour un routeur autonome.
- **eMMC dans le slot microSD** : des adaptateurs existent (Hardkernel, Radxa, SpotPear), mais leur compatibilité avec les Raspberry Pi est incertaine : un utilisateur rapporte qu'un adaptateur Radxa marche sur un Pi 4 mais pas sur un Pi 3A+ (famille proche du Zero 2 W). Rien ne permet de brancher un SSD, SATA ou NVMe derrière le slot SD.

### Fiabilité d'une carte SD

- Les coupures de courant, les écritures répétées (logs, swap) et les cartes de mauvaise qualité ou contrefaites sont les causes principales de panne. Vérifie aussi l'alimentation et le contact de la carte dans son slot.
- **Racine en lecture seule** avec overlay en RAM : réduit l'usure et les corruptions. Les logs disparaissent au redémarrage, donc un envoi des logs à distance aide beaucoup. Sur Void, il faut des `tmpfs` pour `/var/log`, `/tmp` et les dossiers où dhcpcd et dnsmasq écrivent (baux).
- Cartes **High Endurance** ou industrielles (pSLC, SLC) pour un équipement allumé en permanence ; test de capacité réelle avec `f3` (ou H2testw).
- **Image de secours** de la carte (`dd`) conservée ailleurs, plus la configuration versionnée dans git : si la carte meurt, on reflashe et c'est reparti.
- Garder 2 ou 3 cartes de rechange clonées depuis l'image de référence est une stratégie courante, à condition de refaire l'image après chaque changement important.

### 4G

Le guide de la carte Dual-ETH mentionne un module 4G (Cat1) via un connecteur réservé sur la version récente, qui partage les signaux d'un des deux ports USB-A. Je n'ai pas trouvé d'annonce séparée pour ce module. Les produits 4G trouvés sont des cartes à part entière (SpotPear « Zero 4G Cat1 Hub », Geekworm « RPi0 4G Cat1-Hub »), avec 3 ports USB et sans Ethernet. Elles utilisent les mêmes pastilles que la carte Dual-ETH et rendent le port micro-USB de données indisponible : je pense qu'on ne peut pas empiler les deux, mais c'est une déduction, pas une information du fabricant. La 4G Cat1 est limitée (environ 10 Mbps en réception, 5 Mbps en émission d'après les fiches), donc plutôt un lien de secours : une seconde route par défaut avec une `metric` plus élevée (voir 2.4).

---

## Sources

- Void Handbook, configuration réseau : <https://docs.voidlinux.org/config/network/index.html>
- Void Handbook, pare-feu (iptables et nftables) : <https://mirrors.lug.mtu.edu/voidlinux/docs/config/network/firewalls.html>
- Ticket void-packages sur le chemin `/etc/nftables.conf` du service : <https://github.com/void-linux/void-packages/issues/41100>
- Guide de la carte RPi-Zero-Dual-ETH : <https://spotpear.com/index.php/wiki/Raspberry-Pi-Zero-Dual-ETH-RJ45.html>
- Manuel `dhcpcd.conf(5)` : <https://man.netbsd.org/NetBSD-6.1.2/dhcpcd.conf.5>
- Post-installation de Void (socklog, nftables) : <https://computingforgeeks.com/void-linux-post-install-setup/>
- Manuel `dhcpcd.conf(5)` (`denyinterfaces`, `allowinterfaces`) : <https://man.netbsd.org/NetBSD-7.1/dhcpcd.conf.5>
- Manuel de dnsmasq (`--test`, `-k`, `--log-facility`) : <https://unix.com/man-page/linux/8/dnsmasq/>
- ISC DHCP en fin de vie : <https://isc.org/dhcp/> et <https://victor.isc.org/dhcp>
- Manuel de `kea-dhcp4` : <https://man.archlinux.org/man/kea-dhcp4.8.txt>
- Documentation Raspberry Pi, démarrage USB du Zero 2 W (modification) : <https://github.com/raspberrypi/documentation/pull/4247/files>
- Carte 4G SpotPear : <https://spotpear.com/index.php/shop/Raspberry-Pi-Zero-4G-CAT1-USB-HUB-GPS-GNSS-LTE-EA-EU.html>
- Racine en lecture seule sur Raspberry Pi : <https://www.dzombak.com/blog/2024/03/running-a-raspberry-pi-with-a-read-only-root-filesystem/>
