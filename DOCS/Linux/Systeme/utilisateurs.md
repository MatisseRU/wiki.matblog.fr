---
parent: Linux | Système
title: Gestions droits / utilisateurs
layout: default
---

# Création des utilisateurs
### créer un user de manière intéractive
```bash
adduser <username>
```
### créer un user
```bash
useradd <username>
```
### créer un user avec /home/
```bash
useradd -m <username>
```
### créer un user avec home custom
```bash
useradd -b <home path> <username>
```
### créer un user avec date d'expiration
```bash
useradd -e <yyyy-mm-dd> <username>
```
### créer un user avec un shell lors de sa connexion
```bash
useradd -s <chemin shell> <username>
```
### créer un user avec un UID définit
```bash
useradd -u <UID> <username>
```
### créer un user avec un groupe principal custom 
```bash
useradd -g <GID ou GROUPE> <username>
```

# Gestion des utilisateurs
### définir un mot de passe pour un user
```bash
passwd <username>
```
### modifier un user (reprend les mêmes paramètres que pour useradd)
```bash
usermod <username>
```
### ajouter un user dans un groupe
```bash
usermod -aG <GID ou GROUPE> <username>
```

# Suppression des utilisateurs
### supprimer un user (et son /home/)
```bash
userdel -r <username>
```

# Gestion des groupes
### créer un groupe
```bash
groupadd <groupname>
```
### ajouter un user dans un groupe
```bash
usermod -aG <GID ou GROUPE> <username>
```
### supprimer un user d'un groupe
```bash
gpasswd -d <username> <GROUPE>
```

# Listing user et groupes
### lister les utilisateurs
```bash
cat /etc/passwd
```
### lister les groupes
```bash
getent group
```

# Gérer les sudoers
## La commande sudo visudo (/etc/sudoers)
### ajouter un user en administrateur "en dur"
```conf
<username> ALL=(ALL) ALL
```
## Le groupe sudo
### ajouter un user dans le groupe sudo
```bash
usermod -aG sudo <username>
```

# Gérer les droits sur un dossier / fichier
### donner uniquement les droits en lecture
```bash
chmod 444 <chemin>
```
### donner uniquement les droits en écriture
```bash
chmod 222 <chemin>
```
### donner uniquement les droits en exécution
```bash
chmod 111 <chemin>
```
### donner tous les droits de manière récursif
```bash
chmod 777 <chemin dossier> -R
```

# Gérer l'ownership sur un dossier / fichier
### donner l'ownership à un user et un groupe
```bash
chown username:groupname <chemin>
```
### donner l'ownership de manière récursif
```bash
chown username:groupname <chemin> -R
```

