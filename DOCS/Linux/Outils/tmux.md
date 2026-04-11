---
parent: Outils (linux)
title: TMUX
layout: default
---

# Cheatsheet TMUX – Spectre

---

## 1. Gestion des Sessions

| Commande | Raccourci | Description |
|----------|-----------|-------------|
| `tmux new -s nom` | – | Créer une nouvelle session nommée "nom" |
| `tmux ls` | – | Lister toutes les sessions |
| `tmux attach -t nom` | – | Se connecter à la session "nom" |
| `tmux rename-session -t ancien new` | `Ctrl+B $` | Renommer une session |
| `tmux kill-session -t nom` | – | Fermer la session "nom" |
| `tmux kill-server` | – | Fermer toutes les sessions |

---

## 2. Gestion des Fenêtres (dans une session)

| Commande | Raccourci | Description |
|----------|-----------|-------------|
| `Ctrl+B c` | – | Créer une nouvelle fenêtre |
| `Ctrl+B n` | – | Fenêtre suivante |
| `Ctrl+B p` | – | Fenêtre précédente |
| `Ctrl+B w` | – | Lister les fenêtres (interface interactive) |
| `Ctrl+B ,` | – | Renommer la fenêtre actuelle |
| `Ctrl+B &` | – | Fermer la fenêtre actuelle |

---

## 3. Gestion des Panneaux (split screen)

| Commande | Raccourci | Description |
|----------|-----------|-------------|
| `Ctrl+B "` | – | Split horizontal (fenêtre divisée en 2) |
| `Ctrl+B %` | – | Split vertical |
| `Ctrl+B flèches` | – | Changer de panneau |
| `Ctrl+B x` | – | Fermer le panneau actuel |
| `Ctrl+B z` | – | Zoomer/dézoomer sur un panneau |
| `Ctrl+B o` | – | Changer de panneau dans le sens horaire |
| `Ctrl+B q` | – | Afficher les numéros de panneaux (puis taper le numéro) |

---

## 4. Raccourcis Système

| Commande | Raccourci | Description |
|----------|-----------|-------------|
| `Ctrl+B d` | – | Détacher la session (quitter sans fermer) |
| `Ctrl+B ?` | – | Lister tous les raccourcis TMUX |
| `Ctrl+B :` | – | Entrer en mode commande |
| `Ctrl+B [` | – | Entrer en mode copie (pour scroll) |
| `Ctrl+B ]` | – | Coller depuis le buffer |
| `Ctrl+B :` puis `set -g mouse on` | – | Activer le défilement à la souris |

---

## 5. Copier/Coller (Mode Copie)
1. `Ctrl+B [` : Entrer en mode copie.
2. Se déplacer avec flèches ou Page Up/Down.
3. Appuyer sur Espace pour commencer la sélection.
4. Sélectionner du texte, puis appuyer sur Entrée pour copier.
5. `Ctrl+B ]` : Coller le texte copié.

---

## 6. Personnalisation (dans `~/.tmux.conf`)
```bash
# Activer la souris
set -g mouse on

# Raccourci pour détacher : Ctrl+D (au lieu de Ctrl+B d)
bind C-d detach-client

# Lancer tmux automatiquement au démarrage du terminal
if shell "test -z "$TMUX"", "tmux attach -t default" || "tmux new -s default"
```

---

## 7. Astuces Utiles
- Rechercher dans le terminal : `Ctrl+B [` puis `/` (mode recherche).
- Redimensionner un panneau : `Ctrl+B :` puis `resize-pane -L 10` (agrandir de 10 colonnes à gauche).
- Envoyer une commande à tous les panneaux : `Ctrl+B :` puis `send-keys -t :. "ma_commande" C-m`.
- Sauvegarder une session : `tmux capture-pane -p -S -100 > session.txt`.

---

## 8. Commandes Utiles du Mode Commande (`Ctrl+B :`)

| Commande | Description |
|----------|-------------|
| `list-commands` | Lister toutes les commandes |
| `source-file ~/.tmux.conf` | Recharger la config |
| `display-message "Hello"` | Afficher un message |
| `new-window -n "logs"` | Créer une fenêtre nommée "logs" |
| `swap-window -s 1 -t 2` | Échanger les fenêtres 1 et 2 |

---

### Pour aller plus loin
- **TMUX Resurrect** : Sauvegarder/Restore les sessions (`tmux-resurrect`).
- **TMUX Continuum** : Sauvegarde automatique.
- **TMUX Sensible** : Pour une navigation plus intuitive.
