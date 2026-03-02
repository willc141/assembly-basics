# Assembly x86-64 - Dados e memória

## Seção .data: Armazenar na Memória

Até agora usávamos valores embutidos nas instruções. Agora vamos guardar dados (strings, números) na memória RAM usando a seção `.data`.

---

## Programa: Imprimir "Olá, Assembly!"

```assembly
global _start

section .data
    mensagem db "Olá, Assembly!", 10
    tamanho equ $ - mensagem

section .text

_start:
    ; syscall write(1, mensagem, tamanho)
    mov rax, 1              ; syscall write
    mov rdi, 1              ; stdout
    mov rsi, mensagem       ; endereço da string
    mov rdx, tamanho        ; número de bytes
    syscall

    ; syscall exit(0)
    mov rax, 60
    mov rdi, 0
    syscall
```

Salve como `ola.asm`

---

## O que Acontece

| Linha | O que faz |
|-------|-----------|
| `section .data` | Seção de dados inicializados |
| `db "texto", 10` | Define bytes (string + newline ASCII) |
| `equ $ - mensagem` | Calcula tamanho: posição atual - início |
| `mov rsi, mensagem` | Coloca o **endereço** em rsi |
| `syscall (write)` | Escreve rdx bytes a partir do endereço rsi |

---

## Conceitos Chave

### db (Define Byte)
Define bytes na memória. Variações:
- `db`: 1 byte
- `dw`: 2 bytes (word)
- `dd`: 4 bytes (double word)
- `dq`: 8 bytes (quad word)

### $ (Posição Atual)
Representa o endereço atual durante a montagem.
```
tamanho equ $ - mensagem
```
Calcula automaticamente: bytes escritos = posição final - posição inicial

### Rótulo = Endereço
```
mov rsi, mensagem       ; rsi recebe o ENDEREÇO
mov rsi, [mensagem]     ; rsi recebe o CONTEÚDO (primeiros 8 bytes)
```

**Sem colchetes** = endereço (ponteiro)  
**Com colchetes** = conteúdo da memória

### Syscall write (número 1)
Escreve dados no terminal:
- `rdi = 1` (stdout)
- `rsi` = endereço do buffer
- `rdx` = número de bytes

---

## Executar

```bash
nasm -f elf64 ola.asm -o ola.o
ld ola.o -o ola
./ola
```

Resultado: `Olá, Assembly!` aparece na tela.

---

## Comparação com C

```c
#include <unistd.h>

int main() {
    char mensagem[] = "Olá, Assembly!\n";
    write(1, mensagem, 15);
    return 0;
}
```

---

## Exercícios

1. **Mude a mensagem** para imprimir seu nome. Ajuste `db` e recalcule o tamanho automaticamente.

2. **Duas syscalls write**: remova o `10` da string e faça duas syscalls separadas:
   - Uma para a mensagem
   - Outra para um newline

3. **Desafio**: carregue números da memória, some-os e escreva o resultado:
   ```assembly
   mov r10, [numero1]      ; carrega numero1
   mov r11, [numero2]      ; carrega numero2
   add r10, r11            ; soma
   mov [resultado], r10    ; guarda resultado
   ```
   Use `dq` para números de 64 bits e exit com o resultado.
