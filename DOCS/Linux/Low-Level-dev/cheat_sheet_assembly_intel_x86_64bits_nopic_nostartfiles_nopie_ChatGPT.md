---
parent: Low-Level-dev (linux)
title: Final x86_64 Cheat Sheet (Linux 64-bit, Intel syntax) lazy ChatGPT generated
layout: default

---
# x86_64 Cheat Sheet (Linux 64-bit, Intel syntax) lazy ChatGPT generated

---

## 1. Registres x86_64

### Registres généraux (64 bits)

| Registre | Description |
|----------|------------|
| rax | Accumulateur, valeur de retour, syscall number |
| rbx | Général (callee-saved) |
| rcx | Compteur |
| rdx | 3e argument |
| rsi | 2e argument |
| rdi | 1er argument |
| rbp | Base pointer (frame stack) |
| rsp | Stack pointer |
| r8-r15 | Registres supplémentaires |

---

### Sous-registres

| 64 bits | 32 bits | 16 bits | 8 bits |
|--------|--------|--------|--------|
| rax | eax | ax | al / ah |
| rbx | ebx | bx | bl / bh |
| rcx | ecx | cx | cl / ch |
| rdx | edx | dx | dl / dh |

Pour r8-r15 :
- r8 → r8d → r8w → r8b

---

### Registres spéciaux

- rip : instruction pointer
- rflags : flags CPU

---

## 2. Convention d'appel Linux (syscalls)

### Registres utilisés

| Argument | Registre |
|----------|----------|
| syscall number | rax |
| arg1 | rdi |
| arg2 | rsi |
| arg3 | rdx |
| arg4 | r10 |
| arg5 | r8 |
| arg6 | r9 |

---

### Exemple : write

```asm
mov rax, 1      ; syscall write
mov rdi, 1      ; stdout
mov rsi, buffer
mov rdx, length
syscall
```

---

## 3. Gestion de la pile (stack)

### Registres importants

- rsp : sommet de la pile
- rbp : base de la frame

---

### Prologue standard

```asm
push rbp
mov rbp, rsp
sub rsp, 32    ; allocation locale
```

---

### Epilogue

```asm
mov rsp, rbp
pop rbp
ret
```

---

### Variable locale

```asm
mov rax, 42
mov [rbp - 8], rax
```

---

## 4. Allocation et instruction mov

Important : `mov` n’alloue pas de mémoire.

Exemple incorrect si pas de place sur la stack :

```asm
mov [rbp - 8], rax
```

Allocation correcte :

```asm
sub rsp, 32
mov [rbp - 8], rax
```

---

## 5. Segments mémoire

### .data (initialisé)

```asm
.section .data
msg: .asciz "Hello"
msg_len = . - msg
```

---

### .bss (non initialisé)

```asm
.section .bss
buffer: .skip 64
```

---

### Stack (dynamique)

```asm
sub rsp, 32
```

---

## 6. Buffers

### Buffer sur la stack

```asm
lea rsi, [rbp - 32]
```

---

### Buffer dans .bss

```asm
mov rsi, buffer
```

---

## 7. Types de pointeurs

| Type | Taille |
|------|--------|
| byte ptr | 1 byte |
| word ptr | 2 bytes |
| dword ptr | 4 bytes |
| qword ptr | 8 bytes |

---

### Exemple

```asm
mov byte ptr [rax], 1
mov word ptr [rax], 2
mov dword ptr [rax], 3
mov qword ptr [rax], 4
```

---

## 8. Donner un pointeur à une zone mémoire

### Adresse d'une variable globale

```asm
mov rsi, buffer
```

---

### Adresse sur la stack

```asm
lea rsi, [rbp - 32]
```

---

### Différence

- `mov` : charge une valeur
- `lea` : charge une adresse

---

## 9. Exemple minimal (read + write + exit)

```asm
.intel_syntax noprefix
.global _start

.section .bss
buffer: .skip 32

.section .text

_start:
    push rbp
    mov rbp, rsp

    ; read(stdin, buffer, 32)
    mov rax, 0
    mov rdi, 0
    mov rsi, buffer
    mov rdx, 32
    syscall

    ; write(stdout, buffer, rax)
    mov rdx, rax
    mov rax, 1
    mov rdi, 1
    mov rsi, buffer
    syscall

    ; exit(0)
    mov rax, 60
    xor rdi, rdi
    syscall
```

---

## 10. Exemple avec buffer sur la stack

```asm
.intel_syntax noprefix
.global _start

.section .text

_start:
    push rbp
    mov rbp, rsp
    sub rsp, 32

    ; read
    mov rax, 0
    mov rdi, 0
    lea rsi, [rbp - 32]
    mov rdx, 32
    syscall

    ; write
    mov rdx, rax
    mov rax, 1
    mov rdi, 1
    lea rsi, [rbp - 32]
    syscall

    mov rsp, rbp
    pop rbp

    ; exit
    mov rax, 60
    xor rdi, rdi
    syscall
```

---

## 11. Exemple type CTF (inspiré de ton modèle)

```asm
.intel_syntax noprefix
.global _start

.section .data
password: .asciz "secret"
password_len = . - password

success: .asciz "OK\n"
success_len = . - success

fail: .asciz "NO\n"
fail_len = . - fail

.section .bss
input: .skip 32

.section .text

_start:
    push rbp
    mov rbp, rsp

    ; read input
    mov rax, 0
    mov rdi, 0
    mov rsi, input
    mov rdx, 32
    syscall

    ; comparaison simple (1er caractère)
    mov al, byte ptr [input]
    mov bl, byte ptr [password]
    cmp al, bl
    jne wrong

correct:
    mov rax, 1
    mov rdi, 1
    mov rsi, success
    mov rdx, success_len
    syscall
    jmp exit

wrong:
    mov rax, 1
    mov rdi, 1
    mov rsi, fail
    mov rdx, fail_len
    syscall

exit:
    mov rax, 60
    xor rdi, rdi
    syscall
```

---

## 12. Compilation

```bash
gcc -nostdlib -nostartfiles -fno-pic -no-pie file.s -o file
```

---

## 13. Points importants

- Toujours réserver la stack avant d'écrire dessus
- Utiliser `lea` pour obtenir une adresse
- Utiliser `.bss` pour les buffers
- Utiliser `.data` pour les constantes
- rax contient le numéro de syscall et la valeur de retour
- Respecter l’ordre des registres pour les syscalls
- Nettoyer la stack avant de quitter une fonction si rbp est utilisé

---