---
parent: Low-Level-dev (linux)
title: cheat sheet assembleur x_86_64_intel_nolibc CHATGPT
layout: default
---

# Cheat Sheet Assembleur x86_64 (Syntaxe Intel, sans libc) CHATGPT

Compilation recommandée :
```
gcc -nostdlib -nostartfiles -fno-pic -no-pie file.s -o prog
```

Activer la syntaxe Intel dans GAS :
```
.intel_syntax noprefix
```

---

# 1. Sauts conditionnels et flags

## Comparaison
```
cmp rax, rbx        ; calcule rax - rbx et met à jour les flags
```

## Flags principaux
- ZF (Zero Flag) : 1 si résultat = 0
- SF (Sign Flag) : 1 si résultat négatif
- CF (Carry Flag) : retenue (arithmétique non signée)
- OF (Overflow Flag) : overflow signé

## Sauts conditionnels

### Égalité
```
je label            ; ZF=1
jne label           ; ZF=0
jz label            ; alias de je
jnz label           ; alias de jne
```

### Comparaison signée
```
jl label            ; <
jle label           ; <=
jg label            ; >
jge label           ; >=
```

### Comparaison non signée
```
jb label            ; < (below)
jbe label           ; <=
ja label            ; > (above)
jae label           ; >=
```

### Autres
```
jmp label           ; saut inconditionnel
call label          ; appel (push RIP puis jump)
ret                 ; retour (pop RIP)
```

---

# 2. Variables globales (.data)

Section :
```
.section .data
```

## Déclarations (GAS)

### Entiers
```
var8:   .byte 0x41
var16:  .word 0x1234
var32:  .long 0x12345678
var64:  .quad 0x1122334455667788
```

### Tableaux
```
array:  .byte 1,2,3,4
```

### Accès (Intel syntax)
```
mov al, [var8]
mov rax, [var64]
lea rdi, [var8]
```

Avec -no-pie (adresses absolues autorisées) :
```
mov rdi, var8       ; adresse directe
```

---

# 3. Chaînes de caractères globales

## Chaîne simple (terminée par 0)
```
.section .data

msg:
    .asciz "hello"
```

## Sans terminateur
```
msg:
    .ascii "hello"
msg_len = . - msg
```

## Avec saut de ligne
```
msg:
    .asciz "hello\n"
msg_len = . - msg      ; inclut le \0
```

## Longueur (inclut le \0 avec .asciz)
```
msg_len = . - msg
```

## Longueur sans le \0
```
msg_len = . - msg - 1
```

---

# 4. Exemple complet (write syscall, sans libc)

```
.intel_syntax noprefix

.section .data
msg:
    .ascii "Hello\n"
msg_len = . - msg

.section .text
.global _start

_start:
    mov rax, 1          ; syscall write
    mov rdi, 1          ; stdout
    mov rsi, msg        ; adresse (OK avec -no-pie)
    mov rdx, msg_len
    syscall

    mov rax, 60         ; exit
    xor rdi, rdi
    syscall
```

---

# 5. Rappels utiles

## Registres arguments (Linux x86_64)
- rdi, rsi, rdx, rcx, r8, r9

## Instructions utiles
```
cmp a, b
jmp label
call label
ret
lea rax, [var]
mov rax, var        ; adresse absolue (nécessite -no-pie)
```

---

# 6. Sections mémoire

- .text : code
- .data : données initialisées
- .bss  : données non initialisées

Exemple :
```
.section .bss
buffer: .skip 64
```

