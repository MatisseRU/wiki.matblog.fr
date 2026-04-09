---
parent: Outils (linux)
title: Bash
layout: default
---

### 1. Shebang
```bash
#!/bin/bash
```

---

### 2. Types de données
- **Chaînes de caractères** : `"texte"` ou `'texte'`
- **Nombres entiers** : `42` (pas de décimaux, pas de type float)
- **Tableaux** :
  ```bash
  tableau=("val1" "val2" "val3")
  ```
  Accès : `${tableau[0]}`, `${tableau[@]}` (tous les éléments)

---

### 3. Opérations sur les chaînes
- **Concaténation** :
  ```bash
  str1="Hello"
  str2="World"
  result="$str1 $str2"  # "Hello World"
  ```
- **Longueur** :
  ```bash
  echo ${#str1}  # 5
  ```
- **Extraction** :
  ```bash
  echo ${str1:0:2}  # "He" (de l’index 0, 2 caractères)
  ```
- **Remplacement** :
  ```bash
  echo ${str1/e/E}  # "HEllo" (remplace le premier 'e' par 'E')
  echo ${str1//e/E} # "HEllo" (remplace tous les 'e' par 'E')
  ```
- **Suppression** :
  ```bash
  echo ${str1#H}    # "ello" (supprime le premier 'H')
  echo ${str1##He}  # "llo" (supprime le préfixe "He")
  ```

---

### 4. Variables
- Déclaration : `var="valeur"`
- Utilisation : `$var` ou `${var}`

---

### 5. Structures de contrôle
- **If/Else**
  ```bash
  if [ condition ]; then
      # code
  elif [ condition ]; then
      # code
  else
      # code
  fi
  ```
- **Boucles**
  ```bash
  for i in 1 2 3; do
      echo $i
  done
  ```
  ```bash
  while [ condition ]; do
      # code
  done
  ```

---

### 6. Arguments
- `$0` : nom du script
- `$1`, `$2` : arguments 1, 2, etc.
- `$#` : nombre d’arguments
- `$@` : liste de tous les arguments

---

### 7. Entrées/Sorties
- Lire une entrée :
  ```bash
  read -p "Prompt: " var
  ```
- Afficher :
  ```bash
  echo "Texte"
  ```

---

### 8. Exécution
- Rendre exécutable :
  ```bash
  chmod +x script.sh
  ```
- Lancer :
  ```bash
  ./script.sh
  ```
  ou
  ```bash
  bash script.sh
  ```

---

### 9. Gestion des erreurs
- `$?` : code de retour (0 = OK)
- `set -e` : stoppe le script en cas d’erreur

---

### 10. Exemple : Vérifier si l'utilisateur est root
```bash
#!/bin/bash

if [ "$(id -u)" -eq 0 ]; then
    echo "L'utilisateur est root."
else
    echo "L'utilisateur n'est pas root. Certaines commandes peuvent nécessiter des droits administrateur."
    exit 1
fi
```

