---
parent: Void Linux
title: 01 Installation Void musl UEFI ou BIOS legacy
layout: default
---

# Partie 1 : Installation de Void Linux musl

## 1.1 Partitionnement

Deux variantes selon le mode de démarrage de la machine (ou de la VM).

### Variante A : UEFI (table GPT)

```sh
cfdisk /dev/sda
```

- Table de partitions : **gpt**
- Partition 1 : **512 Mo**, type *EFI System* (FAT32, `/boot/efi`)
- Partition 2 : **2 Go**, type *Linux swap*
- Partition 3 : **le reste**, type *Linux filesystem* (EXT4, `/`)

Formatage :

```sh
mkfs.fat -F 32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
```

Montage :

```sh
mount /dev/sda3 /mnt
mount --mkdir /dev/sda1 /mnt/boot/efi
swapon /dev/sda2
```

### Variante B : BIOS Legacy (table MBR / DOS)

Pas de partition EFI nécessaire.

```sh
cfdisk /dev/sda
```

- Table de partitions : **dos**
- Partition 1 : **2 Go**, *Linux swap*
- Partition 2 : **le reste**, *Linux* (EXT4), avec le drapeau **bootable**

```sh
mkswap /dev/sda1
mkfs.ext4 /dev/sda2
mount /dev/sda2 /mnt
swapon /dev/sda1
```

## 1.2 Installation du système de base

Depuis le live ISO **musl** :

```sh
xbps-install -Sy -R https://repo-default.voidlinux.org/current/musl -r /mnt base-system
xgenfstab -U /mnt > /mnt/etc/fstab
xchroot /mnt /bin/bash
```

- `-r /mnt` installe dans la racine cible et non dans le live.
- `xgenfstab -U` génère le `fstab` avec des UUID, à partir de ce qui est monté sous `/mnt`.
- `xchroot` (paquet `xtools`) entre dans le système cible en montant `/dev`, `/proc`, `/sys`, etc.

## 1.3 Configuration dans le chroot

Les commandes suivantes sont à taper **dans le chroot**.

```sh
xbps-install base-devel vim sudo
echo "monvoid" > /etc/hostname
passwd                                   # mot de passe root
useradd -m -G users,audio,video voiduser
passwd voiduser
EDITOR=vim visudo
```

Dans `visudo`, ajouter la ligne :

```
voiduser ALL=(ALL:ALL) ALL
```

### Chargeur de démarrage (GRUB)

**UEFI :**

```sh
xbps-install grub grub-x86_64-efi efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id="Void"
```

**BIOS Legacy (MBR) :**

```sh
xbps-install grub
grub-install --target=i386-pc /dev/sda
```

Dans les deux cas :

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### Réseau minimal : activer dhcpcd

```sh
xbps-install dhcpcd
ln -s /etc/sv/dhcpcd /etc/runit/runsvdir/default/dhcpcd
```

Ce lien symbolique est ce qui « active » le service au prochain démarrage (voir la partie 3 pour le détail du fonctionnement).

## 1.4 Interface graphique (optionnel)

```sh
xbps-install xorg xinit
xbps-install i3 i3status alacritty firefox dmenu
```

`xorg` est un méta-paquet volumineux. Pour du plus léger, `xorg-minimal` existe, mais il faut alors installer soi-même les drivers et outils nécessaires.

## 1.5 Finalisation et redémarrage

Dans le chroot, puis hors du chroot :

```sh
xbps-reconfigure -fa      # dans le chroot
exit                      # sort du chroot
umount -R /mnt
reboot
```

## 1.6 Après le premier démarrage

Connecté avec l'utilisateur, dans `~` :

```sh
echo 'exec i3' >> ~/.xinitrc
startx
```

Au premier lancement, i3 propose un assistant : appuyer sur **Entrée** pour générer la configuration, puis sur **Entrée** à nouveau pour choisir la touche Windows comme touche modificatrice.

## 1.7 Clavier azerty (français)

```sh
vim /etc/rc.conf
```

Ajouter ou modifier la ligne :

```
KEYMAP="fr"
```

Appliquer tout de suite sans redémarrer :

```sh
loadkeys fr
```

Sous X, pour la session en cours :

```sh
setxkbmap fr
```

Pour que ce soit permanent via `~/.xinitrc`, `setxkbmap fr` doit être placé **avant** `exec i3` (tout ce qui suit `exec` n'est jamais exécuté) :

```sh
# ~/.xinitrc
setxkbmap fr
exec i3
```

Alternative permanente, indépendante du `.xinitrc`, dans `/etc/X11/xorg.conf.d/00-keyboard.conf` :

```
Section "InputClass"
    Identifier "keyboard"
    MatchIsKeyboard "yes"
    Option "XkbLayout" "fr"
EndSection
```

### Fichiers de locale

- `/etc/locale.conf` : variable `LANG` du système, par exemple `LANG=fr_FR.UTF-8`.
- `/etc/default/libc-locales` : liste des locales à générer. Ce fichier ne sert que sur **glibc** : décommenter `fr_FR.UTF-8 UTF-8`, puis lancer `xbps-reconfigure -f glibc-locales`. Il n'a pas d'effet sur un système musl.

## 1.8 Créer une copie glibc utilisable avec xchroot

Utile pour lancer sur un système **musl** des logiciels qui ne fonctionnent qu'avec glibc. On installe un second système (glibc) dans un dossier et on y entre avec `xchroot`.

```sh
sudo mkdir -p /path/to/xchrootedVolume/var/db/xbps/keys
sudo cp /var/db/xbps/keys/* /path/to/xchrootedVolume/var/db/xbps/keys/
sudo XBPS_ARCH=x86_64 xbps-install -S -r /path/to/xchrootedVolume \
     -R "https://repo-default.voidlinux.org/current" base-system
sudo xbps-install -S xtools
sudo xchroot /path/to/xchrootedVolume
```

- `XBPS_ARCH=x86_64` force l'architecture **glibc** (sur un hôte musl, l'architecture native serait `x86_64-musl`).
- Les clés sont copiées avant pour éviter les questions d'import de clé.

Dans le chroot glibc :

```sh
xbps-install -S xorg xinit i3 i3status alacritty firefox dmenu vim htop
xbps-reconfigure -fa
```

## 1.9 Afficher une application X du chroot sur l'écran de l'hôte

Le chroot partage le serveur X de l'hôte via son socket.

```sh
ls -la /tmp/.X11-unix/
sudo mkdir -p /path/to/xchrootedVolume/tmp/.X11-unix
sudo mount --bind /tmp/.X11-unix /path/to/xchrootedVolume/tmp/.X11-unix
echo $XAUTHORITY
sudo cp ~/.Xauthority /path/to/xchrootedVolume/home/voiduserg/.Xauthority
sudo xchroot /path/to/xchrootedVolume/
```

Puis, dans le chroot :

```sh
export DISPLAY=:0
export XAUTHORITY=/home/voiduserg/.Xauthority
```

Remarques :

- L'utilisateur `voiduserg` doit exister **dans le chroot** (`useradd -m voiduserg`), et le fichier `.Xauthority` copié doit lui appartenir (`chown`).
- Si `echo $XAUTHORITY` n'affiche rien côté hôte, X utilise `~/.Xauthority` par défaut.
- **Avant d'éteindre**, démonter le socket partagé :

```sh
sudo umount /path/to/xchrootedVolume/tmp/.X11-unix
```