---
parent: Void Linux
title: 04 Environnement chiffré perso
layout: default
---

# Partie 4 : Scripts utilitaires

Quatre scripts, copiés tels quels. Trois servent à créer et gérer un **volume chiffré en cascade** (`cse`, `mse`, `umse`). Le quatrième (`jinse`) automatise l'entrée dans un chroot avec affichage X11 (voir la section 1.9) et n'a pas de lien avec le chiffrement.

| Script | Rôle |
|---|---|
| `cse` | **crée** le volume chiffré en cascade (formate la partition) |
| `mse` | **monte** le volume (ouvre les trois couches puis monte le système de fichiers) |
| `umse` | **démonte** et ferme proprement les trois couches |
| `jinse` | entre dans un chroot en partageant l'affichage X11 de l'hôte |

## 4.1 Volume chiffré en cascade (LUKS2, 3 couches)

### Principe

Trois couches **LUKS2** sont empilées. Chaque couche utilise un algorithme différent et un **en-tête détaché**, c'est-à-dire un fichier séparé de la partition :

```
Partition (/dev/sdb1)
 └─ vol1   AES-XTS       (en-tête : fichier 1)
     └─ vol2   Serpent-XTS   (en-tête : fichier 2)
         └─ vol3   Twofish-XTS   (en-tête : fichier 3)
             └─ ext4, monté sur le point de montage choisi
```

À l'écriture, les données traversent d'abord Twofish, puis Serpent, puis AES avant d'arriver sur le disque. À l'ouverture, l'ordre est donc `vol1`, puis `vol2`, puis `vol3`, et à la fermeture l'ordre inverse.

### Prérequis

```sh
xbps-install cryptsetup
chmod +x cse mse umse
```

Les scripts utilisent `sudo`, `lsblk` et `mountpoint` (util-linux), ainsi que `mkfs.ext4` (e2fsprogs).

### Utilisation

```sh
./cse      # création : DÉTRUIT les données de la partition choisie
./mse      # montage
./umse     # démontage
```

### Points d'attention

- **En-têtes détachés** : la partition elle-même ne contient aucun en-tête LUKS. Sans les **trois fichiers** d'en-tête, les données sont définitivement perdues. Il faut les conserver en plusieurs exemplaires et **hors** du volume chiffré (clé USB, autre machine). Comme ce sont déjà des fichiers, copier ces trois fichiers constitue une sauvegarde.
- **Trois mots de passe** : chaque couche a sa propre phrase de passe, demandée à la création puis à chaque montage.
- **Temps de dérivation de clé** : `cse` demande, par couche, un temps maximal en secondes (Argon2id). Plus il est long, plus l'ouverture est lente, et plus une attaque par force brute coûte cher.
- **Performance** : chaque lecture ou écriture traverse trois chiffrements, donc le débit est nettement inférieur à celui d'une seule couche, surtout sur une machine modeste.
- **Démontage `umse`** : le script utilise `umount -l` (démontage « paresseux »). Si un processus utilise encore des fichiers du volume, le point de montage est détaché mais le périphérique reste occupé, et `cryptsetup luksClose` peut échouer avec « device busy ». À cause de `set -e`, le script s'arrête alors. Mieux vaut fermer les programmes qui utilisent le volume avant de lancer `umse`.

### `cse` : création du volume

```bash
#!/bin/bash
set -euo pipefail

echo "=== Creating a cascaded encrypted volume (AES inner / Serpent middle / Twofish outer) ==="
echo

echo "Partitions detected on the system:"
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
echo

read -rp "Target partition to encrypt (e.g. /dev/sdb1): " TARGET_PART

if [ ! -b "$TARGET_PART" ]; then
    echo "Error: $TARGET_PART is not a valid block device."
    exit 1
fi

echo
echo "WARNING: all data on $TARGET_PART will be destroyed."
read -rp "Confirm? (type YES in uppercase): " CONFIRM
if [ "$CONFIRM" != "YES" ]; then
    echo "Cancelled."
    exit 1
fi

read -rp "Full path to store the AES layer header (vol1): " HEADER1
read -rp "Full path to store the Serpent layer header (vol2): " HEADER2
read -rp "Full path to store the Twofish layer header (vol3): " HEADER3

for HDR in "$HEADER1" "$HEADER2" "$HEADER3"; do
    if [ -e "$HDR" ]; then
        echo "Error: $HDR already exists, choose another path to avoid overwriting an existing header."
        exit 1
    fi
    mkdir -p "$(dirname "$HDR")"
done

# Helper to read a max iteration time in seconds and convert to milliseconds
read_iter_time() {
    local LABEL="$1"
    local SECONDS_INPUT
    while true; do
        read -rp "Max key derivation time for the $LABEL layer, in seconds (e.g. 5): " SECONDS_INPUT
        if [[ "$SECONDS_INPUT" =~ ^[0-9]+$ ]] && [ "$SECONDS_INPUT" -gt 0 ]; then
            echo $(( SECONDS_INPUT * 1000 ))
            return
        else
            echo "Please enter a positive whole number of seconds." >&2
        fi
    done
}

echo
ITERTIME1_MS=$(read_iter_time "AES (vol1)")
ITERTIME2_MS=$(read_iter_time "Serpent (vol2)")
ITERTIME3_MS=$(read_iter_time "Twofish (vol3)")

echo
echo ">>> Formatting the inner layer (AES) on $TARGET_PART..."
sudo cryptsetup luksFormat \
    --type luks2 --cipher aes-xts-plain64 --key-size 512 \
    --hash sha512 --pbkdf argon2id --iter-time "$ITERTIME1_MS" \
    --header "$HEADER1" "$TARGET_PART"

echo ">>> Opening the AES layer (vol1)..."
sudo cryptsetup luksOpen --header "$HEADER1" "$TARGET_PART" vol1

echo ">>> Formatting the middle layer (Serpent) on /dev/mapper/vol1..."
sudo cryptsetup luksFormat \
    --type luks2 --cipher serpent-xts-plain64 --key-size 512 \
    --hash sha512 --pbkdf argon2id --iter-time "$ITERTIME2_MS" \
    --header "$HEADER2" /dev/mapper/vol1

echo ">>> Opening the Serpent layer (vol2)..."
sudo cryptsetup luksOpen --header "$HEADER2" /dev/mapper/vol1 vol2

echo ">>> Formatting the outer layer (Twofish) on /dev/mapper/vol2..."
sudo cryptsetup luksFormat \
    --type luks2 --cipher twofish-xts-plain64 --key-size 512 \
    --hash sha512 --pbkdf argon2id --iter-time "$ITERTIME3_MS" \
    --header "$HEADER3" /dev/mapper/vol2

echo ">>> Opening the Twofish layer (vol3)..."
sudo cryptsetup luksOpen --header "$HEADER3" /dev/mapper/vol2 vol3

echo ">>> Creating the ext4 filesystem on /dev/mapper/vol3..."
sudo mkfs.ext4 /dev/mapper/vol3

echo
echo "=== Done ==="
echo "Encrypted partition : $TARGET_PART"
echo "vol1 header (AES)     : $HEADER1  (iter-time: ${ITERTIME1_MS} ms)"
echo "vol2 header (Serpent) : $HEADER2  (iter-time: ${ITERTIME2_MS} ms)"
echo "vol3 header (Twofish) : $HEADER3  (iter-time: ${ITERTIME3_MS} ms)"
echo
echo "IMPORTANT: back up these three headers elsewhere (cryptsetup luksHeaderBackup),"
echo "losing any of them means permanent loss of the data."
```

### `mse` : montage du volume

```bash
#!/bin/bash
set -euo pipefail

echo "=== Mounting the cascaded encrypted volume ==="
echo

read -rp "Encrypted partition (e.g. /dev/sdb1): " TARGET_PART
read -rp "Path to vol1 header (AES): " HEADER1
read -rp "Path to vol2 header (Serpent): " HEADER2
read -rp "Path to vol3 header (Twofish): " HEADER3
read -rp "Mount point (e.g. /mnt/vol3): " MOUNT_POINT

for HDR in "$HEADER1" "$HEADER2" "$HEADER3"; do
    if [ ! -f "$HDR" ]; then
        echo "Error: header not found at $HDR"
        exit 1
    fi
done

if [ ! -b "$TARGET_PART" ]; then
    echo "Error: $TARGET_PART is not a valid block device."
    exit 1
fi

echo
echo ">>> Opening the AES layer (vol1)..."
sudo cryptsetup luksOpen --header "$HEADER1" "$TARGET_PART" vol1

echo ">>> Opening the Serpent layer (vol2)..."
sudo cryptsetup luksOpen --header "$HEADER2" /dev/mapper/vol1 vol2

echo ">>> Opening the Twofish layer (vol3)..."
sudo cryptsetup luksOpen --header "$HEADER3" /dev/mapper/vol2 vol3

echo ">>> Mounting on $MOUNT_POINT..."
sudo mkdir -p "$MOUNT_POINT"
sudo mount /dev/mapper/vol3 "$MOUNT_POINT"

echo
echo "=== Volume mounted on $MOUNT_POINT ==="
```

### `umse` : démontage et fermeture

```bash
#!/bin/bash
set -euo pipefail

echo "=== Cleanly unmounting the encrypted volume ==="
echo

read -rp "Mount point to unmount (e.g. /mnt/vol3): " MOUNT_POINT

if mountpoint -q "$MOUNT_POINT"; then
    echo ">>> Unmounting $MOUNT_POINT..."
    sudo umount -l "$MOUNT_POINT"
else
    echo "Info: $MOUNT_POINT is not mounted, continuing anyway."
fi

if [ -e /dev/mapper/vol3 ]; then
    echo ">>> Closing the Twofish layer (vol3)..."
    sudo cryptsetup luksClose vol3
else
    echo "Info: vol3 already closed or nonexistent."
fi

if [ -e /dev/mapper/vol2 ]; then
    echo ">>> Closing the Serpent layer (vol2)..."
    sudo cryptsetup luksClose vol2
else
    echo "Info: vol2 already closed or nonexistent."
fi

if [ -e /dev/mapper/vol1 ]; then
    echo ">>> Closing the AES layer (vol1)..."
    sudo cryptsetup luksClose vol1
else
    echo "Info: vol1 already closed or nonexistent."
fi

echo
echo "=== Everything has been cleanly unmounted and closed ==="
```

## 4.2 `jinse` : chroot avec affichage X11

Ce script prend en argument le chemin de la racine du chroot et automatise la section 1.9 : il affiche `DISPLAY` et `XAUTHORITY`, autorise les connexions X locales, monte le socket X11 de l'hôte dans le chroot, copie le fichier `.Xauthority`, puis entre dans le chroot avec `xchroot`.

Utilisation (le script n'a pas de ligne `#!`, donc on le lance avec `bash`) :

```sh
bash jinse /path/to/xchrootedVolume
```

Une fois dans le chroot, il reste à définir les variables, comme en 1.9 :

```sh
export DISPLAY=:0
export XAUTHORITY=/home/voiduserg/.Xauthority
```

Points d'attention :

- `xhost +local:` autorise **tous les utilisateurs locaux** à se connecter à ton serveur X. Pour refermer cet accès après usage : `xhost -local:`.
- Les utilisateurs `voiduser` (hôte) et `voiduserg` (chroot) sont écrits en dur dans le script.
- Le `.Xauthority` copié avec `sudo cp` appartient à root dans le chroot : le donner à l'utilisateur concerné avec `chown`.
- Le script ne démonte pas le socket X11 à la fin. Avant d'éteindre : `sudo umount /path/to/xchrootedVolume/tmp/.X11-unix`.

```bash
if [ -z $1 ]; then
    echo "Please provide the path to the chrooted '/' root"
    exit 1
fi

echo $DISPLAY
echo $XAUTHORITY

sudo xhost +local:
sudo mkdir -p $1/tmp/.X11-unix
sudo mount --bind /tmp/.X11-unix/ $1/tmp/.X11-unix/
sudo cp /home/voiduser/.Xauthority $1/home/voiduserg/
sudo xchroot $1
```
