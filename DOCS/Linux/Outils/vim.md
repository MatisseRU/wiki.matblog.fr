---
parent: Outils (linux)
title: Vim
layout: default
---

# Liste complète des abréviations de commandes dans Vim

---

---

---

## 🔹 **📌 Les Opérateurs (Verbes) de Base**
Ces commandes **agissent sur du texte** et sont souvent suivies d'un **mouvement** (ex: `d`, `y`, `c`).
Elles suivent la syntaxe :
**`[opérateur][nombre][mouvement]`**
(ex: `d2w` = supprime les 2 mots suivants).

| Opérateur | Nom complet | Action | Exemple |
|-----------|-------------|--------|---------|
| `d` | **Delete** | Supprime le texte et le place dans le registre. | `d3j` → Supprime 3 lignes vers le bas. |
| `c` | **Change** | Supprime le texte **et passe en mode insertion**. | `cw` → Change le mot sous le curseur. |
| `y` | **Yank** (Copier) | Copie le texte dans le registre. | `y$` → Copie jusqu'à la fin de la ligne. |
| `>` | **Indent** | Indente le texte (vers la droite). | `>3j` → Indente 3 lignes vers le bas. |
| `<` | **Unindent** | Désindente le texte (vers la gauche). | `<2k` → Désindente 2 lignes vers le haut. |
| `=` | **Auto-indent** | Corrige l'indentation automatiquement. | `=ap` → Réindente le paragraphe. |
| `g~` | **Toggle case** | Inverse la casse (majuscule ↔ minuscule). | `g~w` → Inverse la casse du mot. |
| `gu` | **Lowercase** | Met en minuscules. | `gu2j` → Met 2 lignes en minuscules. |
| `gU` | **Uppercase** | Met en majuscules. | `gUaw` → Met le mot autour du curseur en majuscules. |
| `!` | **Filter** | Passe le texte à une commande shell externe. | `:!sort` → Trie les lignes sélectionnées. |
| `~` | **Toggle case** (caractère sous le curseur) | Inverse la casse du caractère sous le curseur. | `~` → Inverse la casse du caractère. |
| `r` | **Replace** | Remplace **un seul caractère**. | `rX` → Remplace le caractère sous le curseur par `X`. |
| `R` | **Replace mode** | Remplace plusieurs caractères en mode insertion. | `R` puis tape du texte → Écrasement du texte existant. |
| `s` | **Substitute** | Supprime un caractère et passe en mode insertion. | `s` → Remplace 1 caractère. |
| `S` | **Substitute line** | Supprime toute la ligne et passe en mode insertion. | `S` → Remplace toute la ligne. |
| `x` | **Delete char** | Supprime le caractère **sous le curseur**. | `x` → Supprime 1 caractère. |
| `X` | **Delete before** | Supprime le caractère **avant le curseur**. | `X` → Supprime 1 caractère à gauche. |
| `p` | **Paste** | Colle le contenu du registre **après le curseur**. | `p` → Colle après le curseur. |
| `P` | **Paste before** | Colle le contenu du registre **avant le curseur**. | `P` → Colle avant le curseur. |
| `gp` | **Paste and move** | Colle et déplace le curseur après le texte collé. | `gp` → Colle et place le curseur après. |
| `gP` | **Paste before and move** | Colle avant et déplace le curseur avant le texte collé. | `gP` → Colle avant et place le curseur avant. |
| `J` | **Join lines** | Fusionne la ligne courante avec celle du dessous. | `J` → Fusionne 2 lignes. |
| `gJ` | **Join without space** | Fusionne les lignes **sans espace**. | `gJ` → Fusionne sans espace. |

---

---

## 🔹 **📌 Les Mouvements (Noms)**
Ces commandes **déplacent le curseur**. Elles peuvent être **combinées avec des opérateurs** (ex: `d2w`).

### **🔸 Mouvements de base**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `h` | Gauche (1 caractère). | `3h` → 3 caractères à gauche. |
| `j` | Bas (1 ligne). | `5j` → 5 lignes vers le bas. |
| `k` | Haut (1 ligne). | `2k` → 2 lignes vers le haut. |
| `l` | Droite (1 caractère). | `4l` → 4 caractères à droite. |
| `0` | Début de la ligne. | `0` → Va au début. |
| `^` | Premier caractère non vide de la ligne. | `^` → Début de la première word. |
| `$` | Fin de la ligne. | `$` → Va à la fin. |
| `g_` | Dernier caractère non vide de la ligne. | `g_` → Fin de la dernière word. |
| `gg` | Début du fichier. | `gg` → Première ligne. |
| `G` | Fin du fichier. | `G` → Dernière ligne. |
| `5G` | Va à la ligne 5. | `10G` → Ligne 10. |

### **🔸 Mouvements par mots**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `w` | Mot suivant (début du mot). | `3w` → 3 mots vers la droite. |
| `W` | Mot suivant (en ignorant la ponctuation). | `2W` → 2 mots (sans ponctuation). |
| `b` | Mot précédent (début du mot). | `2b` → 2 mots vers la gauche. |
| `B` | Mot précédent (en ignorant la ponctuation). | `B` → Mot précédent (sans ponctuation). |
| `e` | Fin du mot actuel. | `2e` → 2 mots vers la droite (fin). |
| `E` | Fin du mot (en ignorant la ponctuation). | `E` → Fin du mot (sans ponctuation). |
| `ge` | Fin du mot précédent. | `ge` → Fin du mot à gauche. |

### **🔸 Mouvements par paragraphes et sections**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `}` | Paragraphe suivant (séparé par une ligne vide). | `2}` → 2 paragraphes vers le bas. |
| `{` | Paragraphe précédent. | `{` → Paragraphe précédent. |
| `]]` | Section suivante (définie par `}` en C-like). | `]]` → Section suivante. |
| `[[` | Section précédente. | `[[` → Section précédente. |
| `[]` | Début de la section courante. | `[]` → Début de la section. |
| `][` | Fin de la section courante. | `][` → Fin de la section. |

### **🔸 Mouvements dans la fenêtre**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `H` | Haut de la fenêtre (1ère ligne visible). | `H` → Haut de l'écran. |
| `M` | Milieu de la fenêtre. | `M` → Ligne du milieu. |
| `L` | Bas de la fenêtre (dernière ligne visible). | `L` → Bas de l'écran. |
| `zt` | Faire défiler pour placer la ligne courante en haut. | `zt` → Ligne en haut. |
| `zb` | Faire défiler pour placer la ligne courante en bas. | `zb` → Ligne en bas. |
| `zz` | Centrer la ligne courante à l'écran. | `zz` → Centre la ligne. |

### **🔸 Mouvements par caractères spécifiques**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `f{char}` | Aller au prochain `{char}` sur la ligne. | `fx` → Va au prochain `x`. |
| `F{char}` | Aller au précédent `{char}` sur la ligne. | `Fx` → Va au `x` précédent. |
| `t{char}` | Aller avant le prochain `{char}`. | `tx` → Va avant le prochain `x`. |
| `T{char}` | Aller après le précédent `{char}`. | `Tx` → Va après le `x` précédent. |
| `;` | Répéter le dernier `f`, `F`, `t`, ou `T`. | `fx;;` → Va au 3ème `x` suivant. |
| `,` | Répéter le dernier `f`, `F`, `t`, ou `T` en sens inverse. | `fx,fX` → Va au `x` suivant, puis au `X` précédent. |

### **🔸 Mouvements par correspondances**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `%` | Va à la parenthèse/accolade/crochet correspondante. | `%` → Saute à la `}` correspondante. |
| `[(` | Va à la parenthèse ouvrante `(` précédente non appariée. | `[(` → `(` précédente non appariée. |
| `])` | Va à la parenthèse fermante `)` suivante non appariée. | `])` → `)` suivante non appariée. |
| `[{` | Va à l'accolade ouvrante `{` précédente non appariée. | `[{` → `{` précédente. |
| `}]` | Va à l'accolade fermante `}` suivante non appariée. | `}]` → `}` suivante. |

### **🔸 Mouvements par marques**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `ma` | Place une marque `a` à la position courante. | `ma` → Marque la position avec `a`. |
| \`a\` | Va à la ligne et colonne de la marque `a`. | \`a\` → Va à la marque `a`. |
| `'a` | Va à la ligne de la marque `a` (colonne 1). | `'a` → Ligne de `a`. |
| \`\`\` | Va à la position précédente du curseur. | \`\`\` → Retour en arrière. |
| \` \` | Va à la position exacte avant le dernier saut. | \` \` → Retour précis. |
| `''` | Va à la ligne de la position précédente. | `''` → Ligne précédente. |

### **🔸 Mouvements par recherche**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `/motif` | Recherche vers l'avant `motif`. | `/error` → Cherche "error". |
| `?motif` | Recherche vers l'arrière `motif`. | `?error` → Cherche "error" en arrière. |
| `n` | Va à la prochaine occurrence du dernier motif recherché. | `n` → Occurrence suivante. |
| `N` | Va à la précédente occurrence du dernier motif recherché. | `N` → Occurrence précédente. |
| `*` | Recherche le mot sous le curseur vers l'avant. | `*` → Cherche le mot courant. |
| `#` | Recherche le mot sous le curseur vers l'arrière. | `#` → Cherche le mot courant en arrière. |
| `g*` | Comme `*`, mais inclut les motifs partiels. | `g*` → Cherche motif partiel. |
| `g#` | Comme `#`, mais inclut les motifs partiels. | `g#` → Cherche motif partiel en arrière. |

### **🔸 Mouvements dans les mots (objet texte)**

| Mouvement | Description | Exemple |
|-----------|-------------|---------|
| `aw` | Un mot (inclut les espaces autour). | `daw` → Supprime un mot + espaces. |
| `iw` | Mot intérieur (sans les espaces). | `ciw` → Change le mot sous le curseur. |
| `aW` | Un MOT (comme `aw` mais ignore la ponctuation). | `daW` → Supprime un MOT + espaces. |
| `iW` | MOT intérieur (comme `iw` mais ignore la ponctuation). | `ciW` → Change le MOT sous le curseur. |
| `as` | Une phrase (jusqu'au point). | `das` → Supprime une phrase. |
| `is` | Phrase intérieure. | `cis` → Change la phrase. |
| `ap` | Un paragraphe (jusqu'à une ligne vide). | `dap` → Supprime un paragraphe. |
| `ip` | Paragraphe intérieur. | `cip` → Change le paragraphe. |
| `a"` | Texte entre guillemets (inclut les guillemets). | `da"` → Supprime le texte entre `"` + les `"` |
| `i"` | Texte entre guillemets (sans les guillemets). | `ci"` → Change le texte entre `"` |
| `a'` | Comme `a"` mais pour les apostrophes. | `da'` → Supprime le texte entre `'` + les `'` |
| `i'` | Comme `i"` mais pour les apostrophes. | `ci'` → Change le texte entre `'` |
| `a(` | Texte entre parenthèses (inclut les `()`). | `da(` → Supprime le texte entre `()` + les `()` |
| `i(` | Texte entre parenthèses (sans les `()`). | `ci(` → Change le texte entre `()` |
| `a{` | Texte entre accolades (inclut les `{}`). | `da{` → Supprime le texte entre `{}` + les `{}` |
| `i{` | Texte entre accolades (sans les `{}`). | `ci{` → Change le texte entre `{}` |
| `a[` | Texte entre crochets (inclut les `[]`). | `da[` → Supprime le texte entre `[]` + les `[]` |
| `i[` | Texte entre crochets (sans les `[]`). | `ci[` → Change le texte entre `[]` |
| `a<` | Texte entre `<...>` (inclut les `<>`). | `da<` → Supprime le texte entre `<>` + les `<>` |
| `i<` | Texte entre `<...>` (sans les `<>`). | `ci<` → Change le texte entre `<>` |
| `at` | Balise XML/HTML (inclut la balise). | `dat` → Supprime la balise XML/HTML + son contenu. |
| `it` | Contenu d'une balise XML/HTML. | `cit` → Change le contenu de la balise. |

---

---

## 🔹 **📌 Combinaisons Opérateur + Mouvement (Exemples Pratiques)**

| Combinaison | Action |
|-------------|--------|
| `d2w` | Supprime les 2 mots suivants. |
| `c3j` | Change (supprime + édite) les 3 lignes du dessous. |
| `y$` | Copie jusqu'à la fin de la ligne. |
| `dgg` | Supprime depuis la ligne courante jusqu'au début du fichier. |
| `yG` | Copie depuis la ligne courante jusqu'à la fin du fichier. |
| `caw` | Change le mot autour du curseur (y compris les espaces). |
| `ciw` | Change le mot sous le curseur (sans les espaces). |
| `di"` | Supprime le texte entre les guillemets (sans les guillemets). |
| `da"` | Supprime le texte entre les guillemets (y compris les guillemets). |
| `dap` | Supprime le paragraphe autour du curseur. |
| `>3j` | Indente 3 lignes vers le bas. |
| `<2k` | Désindente 2 lignes vers le haut. |
| `gUaw` | Met le mot autour du curseur en majuscules. |
| `gu2j` | Met les 2 lignes du dessous en minuscules. |

---

---

## 🔹 **📌 Commandes en Mode Visuel**

| Commande | Description |
|----------|-------------|
| `v` | Mode visuel (caractère par caractère). |
| `V` | Mode visuel ligne (ligne par ligne). |
| `Ctrl+v` | Mode visuel bloc (rectangle). |
| `o` | Inverse le début et la fin de la sélection. |
| `O` | Comme `o` mais pour les blocs. |
| `aw` | Sélectionne un mot (mode visuel). |
| `ab` | Sélectionne un bloc (entre `(` et `)`). |
| `aB` | Sélectionne un bloc (entre `{` et `}`). |
| `at` | Sélectionne une balise HTML/XML. |
| `it` | Sélectionne le contenu d'une balise HTML/XML. |
| `>` | Indente la sélection. |
| `<` | Désindente la sélection. |
| `~` | Inverse la casse de la sélection. |
| `gU` | Met la sélection en majuscules. |
| `gu` | Met la sélection en minuscules. |
| `d` | Supprime la sélection. |
| `y` | Copie la sélection. |
| `c` | Change la sélection (supprime + édite). |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) Utiles**

| Commande | Description |
|----------|-------------|
| `:w` | Sauvegarde le fichier. |
| `:wq` ou `:x` | Sauvegarde et quitte. |
| `:q` | Quitte Vim. |
| `:q!` | Quitte sans sauvegarder. |
| `:e fichier` | Ouvre `fichier` dans le buffer courant. |
| `:tabe fichier` | Ouvre `fichier` dans un nouvel onglet. |
| `:sp fichier` | Ouvre `fichier` dans une fenêtre divisée horizontalement. |
| `:vsp fichier` | Ouvre `fichier` dans une fenêtre divisée verticalement. |
| `:bn` | Passe au buffer suivant. |
| `:bp` | Passe au buffer précédent. |
| `:bd` | Ferme le buffer courant. |
| `:ls` | Liste tous les buffers ouverts. |
| `:b3` | Passe au buffer numéro 3. |
| `:s/old/new/` | Remplace `old` par `new` sur la ligne courante. |
| `:s/old/new/g` | Remplace toutes les occurrences de `old` par `new` sur la ligne courante. |
| `:%s/old/new/g` | Remplace toutes les occurrences de `old` par `new` dans tout le fichier. |
| `:%s/old/new/gc` | Remplace avec confirmation (`y`/`n`). |
| `:noh` | Désactive le surlignage de la recherche. |
| `:set hlsearch` | Active le surlignage de la recherche. |
| `:set nohlsearch` | Désactive le surlignage de la recherche. |
| `:set incsearch` | Active la recherche incrémentale. |
| `:set wrap` | Active le retour à la ligne automatique. |
| `:set nowrap` | Désactive le retour à la ligne automatique. |
| `:set paste` | Active le mode coller (désactive l'auto-indentation). |
| `:set nopaste` | Désactive le mode coller. |
| `:r fichier` | Insère le contenu de `fichier` à la position du curseur. |
| `:r !commande` | Insère la sortie de `commande`. |
| `:!commande` | Exécute `commande` dans le shell. |
| `:shell` | Ouvre un shell temporaire. |
| `:make` | Exécute `make`. |
| `:copen` | Ouvre la fenêtre des erreurs de compilation. |
| `:cclose` | Ferme la fenêtre des erreurs. |

---

---

## 🔹 **📌 Raccourcis de Mode Insertion**

| Raccourci | Description |
|-----------|-------------|
| `Ctrl+h` | Supprime le caractère avant le curseur. |
| `Ctrl+w` | Supprime le mot avant le curseur. |
| `Ctrl+u` | Supprime toute la ligne avant le curseur. |
| `Ctrl+r` | Insère le contenu d'un registre. |
| `Ctrl+o` | Passe temporairement en mode normal pour une commande. |
| `Ctrl+[` | Quitte le mode insertion (équivalent à `Esc`). |
| `Ctrl+c` | Quitte le mode insertion (comme `Esc`). |
| `Ctrl+v` | Insère le caractère suivant littéralement. |
| `Ctrl+a` | Insère le dernier texte inséré. |
| `Ctrl+@` | Insère le dernier texte inséré et passe en mode insertion. |
| `Ctrl+n` | Complétion de mot (prochain mot du buffer). |
| `Ctrl+p` | Complétion de mot (mot précédent du buffer). |
| `Ctrl+x` | Mode de complétion avancée. |

---

---

## 🔹 **📌 Registres (Classeurs)**

| Registre | Description | Exemple |
|----------|-------------|---------|
| `""` | Registre par défaut (non nommé). | `"ay` → Copie dans le registre `a`. |
| `"0` | Dernier texte yanké (copié). | `"0p` → Colle le dernier yank. |
| `"1` à `"9` | Registres numérotés (1 = dernier suppression, etc.). | `"1p` → Colle la dernière suppression. |
| `"a` à `"z` | Registres nommés (26 lettres). | `"bp` → Colle le contenu du registre `b`. |
| `"+` | Registre du presse-papiers système. | `"+y` → Copie dans le presse-papiers système. |
| `"*` | Registre du presse-papiers primaire. | `"*p` → Colle depuis la sélection souris. |
| `"-` | Registre des petites suppressions. | `"-p` → Colle la dernière petite suppression. |
| `"_` | Registre poubelle. | `"_d` → Supprime sans sauvegarder. |

---

---

## 🔹 **📌 Marques (Bookmarks)**

| Marque | Description | Exemple |
|--------|-------------|---------|
| `ma` | Place une marque `a` à la position courante. | `ma` → Marque la position avec `a`. |
| \`a\` | Va à la ligne et colonne de la marque `a`. | \`a\` → Va à la marque `a`. |
| `'a` | Va à la ligne de la marque `a` (colonne 1). | `'a` → Ligne de `a`. |
| \`\`\` | Va à la position précédente du curseur. | \`\`\` → Retour en arrière. |
| `''` | Va à la ligne de la position précédente. | `''` → Ligne précédente. |
| `Ctrl+o` | Va à l'emplacement précédent dans l'historique. | `Ctrl+o` → Retour en arrière. |
| `Ctrl+i` | Va à l'emplacement suivant dans l'historique. | `Ctrl+i` → Avant dans l'historique. |

---

---

## 🔹 **📌 Macro-commandes**

| Commande | Description |
|----------|-------------|
| `qa` | Commence l'enregistrement dans le registre `a`. |
| `q` | Arrête l'enregistrement. |
| `@a` | Joue la macro du registre `a`. |
| `@@` | Rejoue la dernière macro exécutée. |
| `:reg a` | Affiche le contenu du registre `a`. |

---

---

## 🔹 **📌 Commandes de Fenêtres et Onglets**

| Commande | Description |
|----------|-------------|
| `:split` ou `:sp` | Divise la fenêtre horizontalement. |
| `:vsplit` ou `:vs` | Divise la fenêtre verticalement. |
| `Ctrl+w s` | Divise horizontalement. |
| `Ctrl+w v` | Divise verticalement. |
| `Ctrl+w n` | Nouvelle fenêtre vide. |
| `Ctrl+w q` | Ferme la fenêtre courante. |
| `Ctrl+w o` | Ferme toutes les fenêtres sauf la courante. |
| `Ctrl+w j` | Va à la fenêtre du dessous. |
| `Ctrl+w k` | Va à la fenêtre du dessus. |
| `Ctrl+w h` | Va à la fenêtre de gauche. |
| `Ctrl+w l` | Va à la fenêtre de droite. |
| `Ctrl+w +` | Agrandit la fenêtre. |
| `Ctrl+w -` | Réduit la fenêtre. |
| `Ctrl+w =` | Équilibre la taille des fenêtres. |
| `Ctrl+w >` | Agrandit la fenêtre vers la droite. |
| `Ctrl+w <` | Réduit la fenêtre vers la gauche. |
| `:tabnew` | Ouvre un nouvel onglet. |
| `:tabn` ou `gt` | Va à l'onglet suivant. |
| `:tabp` ou `gT` | Va à l'onglet précédent. |
| `:tabc` | Ferme l'onglet courant. |
| `:tabo` | Ferme tous les onglets sauf le courant. |
| `:tabfirst` | Va au premier onglet. |
| `:tablast` | Va au dernier onglet. |

---

---

## 🔹 **📌 Commandes de Buffer**

| Commande | Description |
|----------|-------------|
| `:ls` ou `:buffers` | Liste tous les buffers ouverts. |
| `:bnext` ou `:bn` | Passe au buffer suivant. |
| `:bprevious` ou `:bp` | Passe au buffer précédent. |
| `:bfirst` | Va au premier buffer. |
| `:blast` | Va au dernier buffer. |
| `:b3` | Va au buffer numéro 3. |
| `:b nom_fichier` | Va au buffer dont le nom contient `nom_fichier`. |
| `:bd` | Ferme le buffer courant. |
| `:bd3` | Ferme le buffer numéro 3. |
| `:bd!` | Ferme le buffer courant sans sauvegarder. |
| `:bw` | Écrit (sauvegarde) le buffer courant. |
| `:ball` | Ouvre tous les buffers dans des fenêtres. |

---

---

## 🔹 **📌 Commandes de Fichiers Externes**

| Commande | Description |
|----------|-------------|
| `:e fichier` | Ouvre `fichier` dans le buffer courant. |
| `:e!` | Recharge le fichier courant (ignore les modifications). |
| `:e +10 fichier` | Ouvre `fichier` à la ligne 10. |
| `:e +/motif fichier` | Ouvre `fichier` à la première occurrence de `motif`. |
| `:r fichier` | Insère le contenu de `fichier` à la position du curseur. |
| `:r !commande` | Insère la sortie de `commande`. |
| `:w fichier` | Sauvegarde le buffer courant dans `fichier`. |
| `:w >> fichier` | Ajoute le buffer courant à la fin de `fichier`. |
| `:!commande` | Exécute `commande` dans le shell. |
| `:shell` | Ouvre un shell temporaire. |
| `:cd chemin` | Change le répertoire courant. |
| `:pwd` | Affiche le répertoire courant. |

---

---

## 🔹 **📌 Commandes de Substitution (`:s`)**

| Commande | Description |
|----------|-------------|
| `:s/old/new/` | Remplace la première occurrence de `old` par `new` sur la ligne courante. |
| `:s/old/new/g` | Remplace toutes les occurrences de `old` par `new` sur la ligne courante. |
| `:%s/old/new/g` | Remplace toutes les occurrences de `old` par `new` dans tout le fichier. |
| `:%s/old/new/gc` | Remplace avec confirmation (`y`/`n`/`a`/`q`/`l`). |
| `:10,20s/old/new/g` | Remplace de la ligne 10 à la 20. |
| `:'<,'>s/old/new/g` | Remplace dans la sélection visuelle. |
| `:s/old/new/i` | Remplace sans tenir compte de la casse. |
| `:s/old/new/I` | Remplace en tenant compte de la casse. |
| `:s/\<word\>/new/g` | Remplace uniquement le mot entier `word`. |
| `:s/^old/new/` | Remplace `old` uniquement au début de la ligne. |
| `:s/old$/new/` | Remplace `old` uniquement à la fin de la ligne. |
| `:s/\d\+/0/g` | Remplace tous les nombres par `0`. |
| `:s/  */ /g` | Remplace plusieurs espaces par un seul. |
| `:s/	/    /g` | Remplace les tabulations par 4 espaces. |

---

---

## 🔹 **📌 Commandes de Recherche et Remplacement Avancées**

| Commande | Description |
|----------|-------------|
| `/motif` | Recherche `motif` vers l'avant. |
| `?motif` | Recherche `motif` vers l'arrière. |
| `n` | Occurrence suivante du dernier motif. |
| `N` | Occurrence précédente du dernier motif. |
| `*` | Recherche le mot sous le curseur vers l'avant. |
| `#` | Recherche le mot sous le curseur vers l'arrière. |
| `g*` | Comme `*`, mais inclut les motifs partiels. |
| `g#` | Comme `#`, mais inclut les motifs partiels. |
| `:vimgrep/motif/ **/*` | Recherche `motif` dans tous les fichiers du répertoire courant. |
| `:copen` | Ouvre la liste des résultats de `:vimgrep`. |
| `:cnext` | Va au résultat suivant dans `:vimgrep`. |
| `:cprevious` | Va au résultat précédent dans `:vimgrep`. |
| `:set hlsearch` | Active le surlignage des résultats de recherche. |
| `:set nohlsearch` | Désactive le surlignage. |
| `:noh` | Désactive temporairement le surlignage. |
| `:set incsearch` | Active la recherche incrémentale. |
| `:set ignorecase` | Ignore la casse dans les recherches. |
| `:set smartcase` | Ignore la casse sauf si majuscules. |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) pour les Fichiers Multiples**

| Commande | Description |
|----------|-------------|
| `:args fichier1 fichier2` | Définit une liste de fichiers à éditer. |
| `:next` ou `:n` | Passe au fichier suivant dans `:args`. |
| `:previous` ou `:N` | Passe au fichier précédent dans `:args`. |
| `:first` | Va au premier fichier dans `:args`. |
| `:last` | Va au dernier fichier dans `:args`. |
| `:argdo %s/old/new/g` | Applique une commande à tous les fichiers dans `:args`. |
| `:drop fichier` | Ouvre `fichier` et ajoute à `:args`. |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) pour les Sessions**

| Commande | Description |
|----------|-------------|
| `:mksession fichier.vim` | Sauvegarde la session courante dans `fichier.vim`. |
| `:source fichier.vim` | Charge une session sauvegardée. |
| `:mks! fichier.vim` | Sauvegarde la session en écrasant `fichier.vim`. |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) pour les Options**

| Commande | Description |
|----------|-------------|
| `:set option` | Active `option`. |
| `:set nooption` | Désactive `option`. |
| `:set option!` | Inverse `option`. |
| `:set option=valeur` | Définit `option` à `valeur`. |
| `:set all` | Affiche toutes les options et leurs valeurs. |
| `:set option?` | Affiche la valeur de `option`. |

**Exemples d'options utiles :**

| Option | Description | Valeur par défaut |
|--------|-------------|-------------------|
| `number` | Affiche les numéros de lignes. | `off` |
| `relativenumber` | Affiche les numéros relatifs. | `off` |
| `tabstop` | Largeur d'une tabulation. | `8` |
| `shiftwidth` | Nombre d'espaces pour l'indentation. | `8` |
| `expandtab` | Remplace les tabulations par des espaces. | `off` |
| `autoindent` | Copie l'indentation de la ligne précédente. | `off` |
| `smartindent` | Indentation intelligente pour les langages. | `off` |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) pour les Abréviations**

| Commande | Description | Exemple |
|----------|-------------|---------|
| `:ab mot expansion` | Définit une abréviation. | `:ab asap as soon as possible` |
| `:ab` | Liste toutes les abréviations. |
| `:unab mot` | Supprime l'abréviation `mot`. |
| `:unab *` | Supprime toutes les abréviations. |
| `:iab mot expansion` | Abréviation uniquement en mode insertion. | `:iab @@ user@example.com` |

---

---

## 🔹 **📌 Commandes de Mode Commande (`:`) pour les Mappings (Raccourcis)**

| Commande | Description | Exemple |
|----------|-------------|---------|
| `:map touche commande` | Crée un mapping dans tous les modes. | `:map <F2> :w<CR>` |
| `:nmap touche commande` | Crée un mapping en mode normal. | `:nmap <Leader>w :w<CR>` |
| `:imap touche commande` | Crée un mapping en mode insertion. | `:imap <C-s> <Esc>:w<CR>a` |
| `:vmap touche commande` | Crée un mapping en mode visuel. | `:vmap <Leader>y "+y` |
| `:nnoremap touche commande` | Crée un mapping non récursif en mode normal. | `:nnoremap <Leader>t :tabnew<CR>` |
| `:inoremap touche commande` | Crée un mapping non récursif en mode insertion. | `:inoremap <C-s> <Esc>:w<CR>a` |
| `:vnoremap touche commande` | Crée un mapping non récursif en mode visuel. | `:vnoremap <Leader>p "+p` |
| `:unmap touche` | Supprime un mapping. | `:unmap <F2>` |
| `:mapclear` | Supprime tous les mappings. |

---

---

## 🔹 **📌 Résumé des Modes de Vim**

| Mode | Description | Comment y entrer | Comment en sortir |
|------|-------------|------------------|-------------------|
| **Normal** | Mode par défaut (navigation, édition). | `Esc` ou `Ctrl+[` | N/A |
| **Insertion** | Mode pour insérer du texte. | `i`, `a`, `I`, `A`, `o`, `O`, `s`, `S`, `R` | `Esc` ou `Ctrl+[` |
| **Visuel** | Mode pour sélectionner du texte. | `v` (caractère), `V` (ligne), `Ctrl+v` (bloc) | `Esc` ou `Ctrl+[` |
| **Commande** | Mode pour entrer des commandes. | `:` | `Esc` ou `Ctrl+[` |
| **Recherche** | Mode pour entrer un motif de recherche. | `/` ou `?` | `Esc` ou `Ctrl+[` |

---

---

## 🔹 **🎯 Bonnes Pratiques et Astuces**

1. **Combiner les opérateurs et mouvements** :
   - `d2w` → Supprime 2 mots.
   - `c3j` → Change 3 lignes vers le bas.

2. **Utiliser les objets texte** :
   - `diw` → Supprime le mot sous le curseur (sans espaces).
   - `ci"` → Change le texte entre guillemets.

3. **Macros pour automatiser** :
   - Enregistre une macro avec `qa`, puis joue-la avec `@a`.

4. **Registres pour copier/coller** :
   - `"+y` → Copie dans le presse-papiers système.

5. **Recherche et remplacement** :
   - `:%s/old/new/gc` → Remplace avec confirmation.

6. **Personnaliser ton `.vimrc`** :
   ```vim
   set number relativenumber
   set tabstop=4 shiftwidth=4 expandtab softtabstop=4
   set hlsearch incsearch ignorecase smartcase
   ```

---

---

## 🔹 **📚 Ressources pour Aller Plus Loin**

- [Vim Adventure](https://vim-adventures.com/)
- [OpenVim](https://www.openvim.com/)
- [Vim Golf](http://www.vimgolf.com/)
- [Vim Cheat Sheet](https://vim.rtorr.com/)
- `:help` dans Vim
