---
parent: Système (linux)
title: Utilitaires Systemd
layout: default
---

# Lecture des logs
### ouvrir en continu de manière tail -f
```bash
journalctl -f
```
### filtrage par service
```bash
journalctl -u <service>
```
### accéder aux logs de l'utilisation de sudo (utiler pour checker l'accès à root)
```bash
journalctl -t sudo
```
### filtrage par pid
```bash
journalctl _PID=1
```
### filtrage par programme (ex: sshd)
```bash
journalctl /usr/sbin/sshd
```
### filtrage par niveau de log
```bash
journalctl -p err
journalctl -p info
```
### filtrage par date
```bash
journalctl --since "2025-01-01 21:31:00"
journalctl --until "2025-12-24 00:00:00"
journalctl --since yesterday
journalctl --since "10 minutes ago"
```
### depuis le démarrage du système
```bash
journalctl -b
```
# Paramétrage des logs
## /etc/systemd/journald.conf
### taille maximum du journal de log
```conf
SystemMaxUse=500M
```
### taille maximum d'un fichier de log
```conf
SystemMaxFileSize=100M
```
### ne pas conserver les logs (rendre les logs volatiles)
```conf
[Journal]
Storage=volatile
```
