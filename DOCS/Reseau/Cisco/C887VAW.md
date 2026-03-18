---
parent: Cisco
title: C887VAW Déployer un réseau Wifi (avec interface pont)
layout: default
---
R301 TP3

# Objectif: Créer un réseau Wifi avec un routeur Cisco C887VAW
## Introduction: Comment fonctionne le routeur et l’AP ?
Ce routeur est particulier, ce n’est pas qu’une seule machine complète, c’est un routeur/switchL3 Cisco sur lequel est greffé un module Access Point Wifi relié par un bridge interne configurable.

D’où le schéma fourni dans le TP:
![schema du routeur Cisco](/assets/C887VAW.png)

Pas de panique j’expliquerai tout en détail dans la partie “configuration de base”. Ce n’est qu’une introduction.
Ce schéma représente l’entièreté de l’équipement Cisco C887VAW, il y a la partie routeur et la partie AP (remarque: sur le schéma, on voit du dualband, 2.4GHz et 5GHz, or, il n’y a qu’une bande 2.4GHz.).

***Ce qu’il faut retenir de ce schéma ET ce qu’il faut faire:***
- Reset le routeur et l’AP
- Configurer le Vlan1 sur la partie routeur du Cisco pour le brancher ensuite sur le vlan 800 de l’IUT.
- Activer l’interface Web (http server) du routeur pour configurer la partie AP (ne le faites pas en ligne de commandes, c’est chiant… et il faut être sûr des commandes et de l’ordre dans lequel les entrer…)
- Configurer le bridge interne pour faire communiquer la partie AP (Wifi) et la partie Réseau (Routeur).
- Configurer un SSID ainsi que la clef WPA pour pouvoir émettre un réseau Wifi fonctionnel (le bridge interne, une fois configuré, va router tous les paquets du réseau wifi vers le vlan associé au SSID que vous avez créé, pas de panique j’explique après.)
- Cabler le Vlan 800 sur un port du switch. Juste en cablant, le DHCP du réseau Wifi (qu’on aurait dû faire en temps normal) sera assuré par le Vlan 800 du réseau de l’IUT. Idem pour les machines connectées sur un port du routeur.


## Etape 1: Reset le router ET l’AP.
### **Reset du routeur:**
Branchez le port console, configurez Minicom, puis allumez le routeur.

Au même moment où vous l’allumez, spammez cette combinaison de touche:  
`CTRL + A ; relacher ; F` <= afin d'envoyer un "BREAK" au routeur.

Jusqu’à voir le terminal:  
`rom1>`

tappez les commandes:  
`rom1> confreg 0x2142`  
`rom2> reset`

Le routeur va redémarrer.


### **Reset de l’AP:**
Une fois connecté en console sur le routeur (après le reset et redémarrage), tappez cette commande pour reset l’AP:  
`router> en`  
`router# service-module wlan-ap 0 reset`

## **Etape 2: Configuration de base**
Tout d’abord, il faut rendre le bridge interne fonctionnel (celui entre la partie routeur du cisco et la partie AP du cisco)

### **Pour commencer faites:**
`router> en`  
`router# show ip int br`

Il y a 2 interfaces à configurer:
- si vous regardez le schéma (et le résultat de la commande), il y a l’interface wlan-ap0. Cette interface sert de “port consol” pour configurer en ligne de commande l’AP, il est crucial de commencer par le configurer pour pouvoir poursuivre la suite du TP.
- sur le schéma, il y a la deuxième interface à configurer qui ne figure pas de manière explicite: grosso modo, le lien “L2” sur le schéma est appelé BVI sur la partie AP, elle n’apparait pas dans le résultat de la commande mais, pas de panique, une fois que vous serez connecté à la partie AP elle figurera.

### **Que faire ?**
Perso quand j’ai fait le TP j’ai pris la plage IP privée suivante:  
192.168.0.0/24 -> réseau interne perso  
192.168.255.1/24 -> interface wlan-ap0 (“port consol” de l’AP)

192.168.0.1/24 -> interface vlan 1  
192.168.0.2/24 -> interface de la BVI de l’AP (doit être sur le même réseau que le vlan 1, c’est sur cette adresse que réside le serveur http de la partie AP du Cisco)

### **Aide commandes à réaliser:**
`router> en`  
`router# conf t`  
`router# interface wlan-ap 0`  
`router# ip addr <IP-PORT-CONSOLE> <MASQUE-SOUS-RESEAU>`  
`router# no shutdown`

Maintenant il faut basculer sur l’AP pour attribuer une IP à la BVI (le bridge interne)

### **Que faire ? + Aide:**
`router# service-module wlan-ap 0 session`  
Le mdp par défaut est Cisco.

`ap> en`  
`ap# show ip int br`  
Là vous allez voir l’interface BVI.

`ap# conf t`  
`ap# int BVI1`  
`ap# ip addr <IP-DE-L’AP> <MASQUE-SOUS-RESEAU>`  
`ap# no sh`  
Maintenant le bridge est configuré. Il faut activer l’interface WEB.

### **Config de l’interface Web de l’AP:**
Créez un user qui aura les droits d’accéder à l’interface Web:

`ap# conf t`  
`ap# username cisco_ap privilege 15 secret Progtr00`  
Ne perdez pas l’identifiant et le mdp car il est crucial pour la suite.

Activez l’interface Web:  
`ap# conf t`  
`ap# ip http server`  
`ap# ip http secure-server`  
`ap# ip http authentication local`

### **Config de l’interface vlan 1 sur le routeur:**
Rebasculez sur la console du routeur avec cette combinaison de touche:  
`CTRL + SHIFT + 6 ;  relâcher puis ; X` <= faut le savoir hein ! ;)

Pour rebasculer dans l’AP il suffit d’appuyer sur ENTER dans la console du routeur sans aucune commande tapée.


### **Aide config interface vlan 1:**
`router> en`  
`router# conf t`  
`router# int vlan 1`  
`router# ip addr <IP-DE-VOTRE-VLAN-1> <MASQUE-SOUS-RESEAU>`  
`router# no sh`

`router# conf t`  
`router# int range f0 - 3`  
`router# switchport mode access`  
`router# switchport access vlan 1`  
`router# no sh`

## **Etape 3: Configuration du réseau Wifi (SSID, WPA, BEACON FRAMES):** ##
Créez une VM en bridge sur la 2ème carte réseau puis débranchez le câble du port sur le mur de votre poste XX.02 et branchez-le sur un des ports de votre routeur.

Connectez vous avec un navigateur (Firefox en désactivant le Proxy) sur l’adresse de l’AP, vous devriez avoir ceci:
![Network Interfaces](/assets/accueuil_ap.jpg)

### **Le bridge vers le vlan 1 du routeur:**
Allez dans: NETWORK INTERFACES -> Service Set Identifiers (SSIDs) -> Define VLANs  
et créez le **Vlan 1** et **cochez bien Native VLAN** et clickez sur **APPLY**

### **Le système de chiffrement:**
Allez dans: SECURITY -> et malheureusement je n’ai plus le cheminement exact mais il faut **activer le système de chiffrement WPA et AES…**

Cela dit j’ai l’image de ce qu’il faut faire:
![Chiffrement](/assets/AP_CHIFFREMENT.jpg)

### **Configuration de votre SSID (votre réseau Wifi: nom, bridge vers un vlan, etc…):**
Retournez dans: NETWORK INTERFACES -> Service Set Identifiers (SSIDs) et mettez ces paramètres:
![SSID](/assets/AP_SSID.jpg)

Mettez un SSID personnalisé, un peu la flemme de voir mon nom circuler… ;)  
(et cochez bien l'interface Radio)

### **Le chiffrement WPA et la configuration du mot de passe lors de la connexion:**
![WPA](/assets/AP_WPA.jpg)
Dans PRE-SHARED-KEY entrez un mot de passe perso pour quand les gens vont se connecter à votre réseau Wifi.

### **Le BEACON (pour que votre réseau Wifi soit visible):**
![SSID](/assets/AP_SSID_2.jpg)

Et voilà, vous avez votre réseau Wifi fonctionnel, normalement … :)  
Il est fonctionnel mais vous n’avez pas encore de réseau, donc pas de panique, c’est normal que votre téléphone / PC ne peut pas se connecter, le réseau doit être juste visible et accessible en Wifi, c’est le VLAN 800 du réseau de l’IUT qui va gérer le DHCP.


## **Etape 4: Relier le réseau Local au VLAN 800 (ou sur tout autre réseau)**
Cette étape est la plus simple, il suffit de tirer un cable depuis le switch dans la baie de brassage vers un port XX.03 (XX = votre numéro de poste) et tirer un autre cable depuis le port XX.03 du mur à côté de votre PC pour le brancher sur un des ports FastEthernet de votre switch.

Et voilà, fini le TP. Bon courage pour la capture de trame ;)

[Mon compte-rendu en PDF](/assets/R301_TP3_Matisse_Rudiger_Vieillame_Quentin_RT2_A2.pdf)


***Tout est fait main, pas d'IA. Grâce à Just-The-Docs, un thème Jekyll.***
{: .label .label-red}
