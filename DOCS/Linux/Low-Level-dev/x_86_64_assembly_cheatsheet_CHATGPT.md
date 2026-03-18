---
parent: Low-Level-dev (linux)
title: x86_64 assembly cheat sheet CHATGPT
layout: default
---

# x86_64 Assembly Linux Cheat Sheet CHATGPT

## 1 Types de données mémoire (`ptr`)

| Syntaxe | Taille | Usage / Registres |
|---------|--------|-----------------|
| `byte ptr`  | 1 octet  | AL, BL, CL… (ASCII, char) |
| `word ptr`  | 2 octets | AX, BX… (short 16 bits) |
| `dword ptr` | 4 octets | EAX, EBX… (int 32 bits) |
| `qword ptr` | 8 octets | RAX, RBX… (long 64 bits / pointeur) |
| `oword ptr` | 16 octets | XMM registers |
| `yword ptr` | 32 octets | YMM registers (AVX) |
| `zword ptr` | 64 octets | ZMM registers (AVX-512) |

> ⚠ Toujours préciser la taille lors de l’accès mémoire. Le registre seul ne suffit jamais pour `[mem]`.

---

## 2 Stack (`rsp`) et `[rsp]`

| Expression | Signification |
|------------|---------------|
| `rsp`      | Adresse actuelle du sommet de la stack |
| `[rsp]`    | Valeur stockée à l’adresse `rsp` |
| `[rsp+offset]` | Valeur à l’adresse décalée |

**Bonnes pratiques :**
- Réserver de l’espace avant usage : `sub rsp, N`
- Restaurer après usage : `add rsp, N`
- Passer **l’adresse (`rsp`)** aux syscalls, jamais la valeur `[rsp]`.
- attention à l'alignement de la stack sur 16 bytes !

---

## 3 Syscalls Linux x86_64

| Argument | Registre |
|----------|----------|
| 1        | rdi      |
| 2        | rsi      |
| 3        | rdx      |
| 4        | r10      |
| 5        | r8       |
| 6        | r9       |
| retour   | rax      |

**Exemples :**

```asm
; write(1, rsp, 2)
mov rax, 1
mov rdi, 1
mov rsi, rsp
mov rdx, 2
syscall

; exit(0)
mov rax, 60
xor rdi, rdi
syscall
```

---

## 4 Manipulation de la stack

```asm
sub rsp, 16                ; réserver 16 octets
mov byte ptr [rsp], 0x41   ; 'A'
mov byte ptr [rsp+1], 0x0A ; '\n'
mov rsi, rsp               ; adresse de la stack pour write
mov rdx, 2
syscall
add rsp, 16                ; restaurer stack
```

- Pas besoin de buffer `.data` pour de petites chaînes temporaires
- Écrire octet par octet ou qword entier selon le besoin

---

## 5 Erreurs fréquentes et bonnes pratiques

| Erreur vue | Pourquoi c’est faux | Correction |
|------------|------------------|-----------|
| `mov byte ptr [rsp+1], 0x8` | 0x8 = backspace, pas retour ligne | `0x0A` pour `\n` |
| `lea rsi, [rip]` pour stack | `[rip]` = adresse code, pas buffer | `mov rsi, rsp` |
| `mov rsi, [rsp]` | `[rsp]` = valeur, pas adresse → write lit mauvaise zone | `mov rsi, rsp` |
| `mov qword ptr [rsp], 0x41` puis `mov qword ptr [rip+buff], rax` | Écrit 8 octets, remplit buffer avec 0s | Utiliser `byte ptr` ou qword entier complet |

---

## 6 Logique générale stack / mémoire

1. **Réserver** → `sub rsp, N`
2. **Écrire** → `mov byte/dword/qword ptr [rsp+offset], valeur`
3. **Passer adresse** → `mov rsi, rsp` pour write / syscall
4. **Syscall** → `mov rax, n`, `mov rdi,rsi…`, `syscall`
5. **Restaurer stack** → `add rsp, N`

---

## 7 Exemple final correct

```asm
.intel_syntax noprefix
.global _start

.section .text
_start:
    sub rsp, 16
    mov byte ptr [rsp], 0x41
    mov byte ptr [rsp+1], 0x0A

    mov rax, 1
    mov rdi, 1
    mov rsi, rsp
    mov rdx, 2
    syscall

    add rsp, 16

    mov rax, 60
    xor rdi, rdi
    syscall
```

- Affiche "A\n"
- Stack restaurée
- Pas de `.data` nécessaire

---

## 8 Diagramme visuel – stack et valeurs

```
Stack (top -> bottom, rsp pointé en haut)

rsp -> +0 : 0x41        ; 'A'
       +1 : 0x0A        ; '\n'
       +2 : ...         ; reste réservé (16 octets au total)
       +3 : ...
       ...
       +15: ...

Instruction:
mov rsi, rsp   ; write prend l'adresse de la stack (buffer)
mov rdx, 2     ; longueur = 2 octets

Write() lit :
[A][\n] -> affichage sur stdout
```

- `rsp` = adresse du sommet de la stack
- `[rsp]` = valeur à cette adresse
- On peut écrire octet par octet ou en qword entier

---

# Ressources utiles

- [System V Application Binary Interface AMD64 Architecture Processor Supplement](https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.99.pdf)
- [Syscall](https://devops-insider.mygraphql.com/zh-cn/latest/kernel/syscall/syscall.html)
- [Syscall Table](https://devops-insider.mygraphql.com/zh-cn/latest/kernel/syscall/syscall-table.html)
- [What are BYTE PTR, WORD PTR, DWORD PTR and QWORD PTR directives in x86 and x64 assembly?](https://www.thesecuritybuddy.com/reverse-engineering/what-are-byte-ptr-word-ptr-dword-ptr-and-qword-ptr-directives-in-x86-and-x64-assembly/)
- [x86 Architecture](https://www.dbgtech.net/windbghelp/hh/debugger/t09_arch_x86_689f0355-a582-450a-bd6c-f17c3167882f.xml.htm)

