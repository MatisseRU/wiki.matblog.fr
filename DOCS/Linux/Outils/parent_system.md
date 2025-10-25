---
parent: Outils (linux)
title: Vim
layout: default
---

# Les déplacements
### Monter
```vim
k
```
### Descendre
```vim
j
```
### Gauche
```vim
h
```
### Droite
```vim
l
```

# Les commandes (mode normal)
Les commandes sont "accumulables" on peut écrire par exemple d3w pour supprimer 3 mots de la ligne (delete 3 words)
## Naviguation
### aller au mot suivant
```vim
e
```
### aller tout en haut du fichier
```vim
G
```
### quitter sans sauvegarder
```vim
:q
```
### quitter sans sauvegarder forcé
```vim
:q!
```
### sauvegarder
```vim
:w
```
### sauvegarder dans un nouveau fichier
```vim
:w fichier.txt
```
### afficher quel fichier et à quelle ligne on se trouve
```vim
CTRL + c  CTRL + g
```
### aller à une ligne (ex: ligne 7)
```vim
:7
```
### rechercher un mot / phrase / expression
```vim
/arbre
```
### exécuter à nouveau la recherche
```vim
n
```
### exécuter à nouveau la recherche mais en arrière
```vim
N
```
### aller à la parenthèse fermante/ouvrante correspondante
```vim
%
```
### exécuter une commande externe (exemple: ls)
```vim
:!ls
```

## Editions
### sauvegarder et quitter
```vim
:wq
```
### effacer un caractère
```vim
x
```
### effacer un mot
```vim
dw
```
### effacer une ligne
```vim
dd
```
### effacer à partir de la ligne jusqu'à la fin de la ligne
```vim
d$
```
### coller après un effacement (CTRL + x  ;  CTRL + v)
```vim
dd (ou dx ou dw etc...)
p
```
### annuler un changement
```vim
u
```
### remplacer un caractère (ici lettre "a")
```vim
ra
```
### remplacer (aka changer) jusqu'à la fin du mot
```vim
ce
(EDITING)
arbre
<ECHAP>
```
### remplacer (aka changer) jusqu'à la fin de la ligne
```vim
c$
j'aime les fruits au sirop
<ECHAP>
```
### substituer 2 mots dans la ligne
```vim
:s/arbre/pommier
```
### substituer tous les mots dans la ligne
```vim
:s/arbre/pommier/g
```
### ajouter le contenu d'un fichier externe
```vim
:r fichier.txt
```
### ajouter le résultat d'une commande externe (exemple: ls)
```vim
:r !ls
```
### ajouter une ligne en dessous et passer en édition
```vim
o
```
### ajouter une ligne au dessus et passer en édition
```vim
O
```

# Les commandes (mode visuel et normal)
### passer en mode visuel
```vim
v
```
### copier
```vim
y
```
### coller
```vim
p
```
### copier un mot
```vim
yw
```
### copier une ligne
```vim
yd
```
### copier une ligne à partir du curseur
```vim
y$
```

