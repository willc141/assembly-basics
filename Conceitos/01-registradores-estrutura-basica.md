# Assembly x86-64 - Guia Básico

## O que é Assembly?

Assembly é uma linguagem de baixo nível que corresponde direto às instruções do processador. Cada comando faz uma coisa simples: mover dados, somar, desviar fluxo. Você controla tudo, registradores, memória, pilha...

Usaremos **x86-64** (processadores Intel/AMD modernos), **sintaxe Intel** (mov destino, fonte) e **Linux com NASM + ld**.

---

## Primeiro Programa: Somar 2 + 3

```assembly
global _start

section .text

_start:
    mov rax, 60      ; syscall exit
    mov rdi, 2       ; primeiro número
    mov rsi, 3       ; segundo número
    add rdi, rsi     ; rdi = rdi + rsi = 5
    syscall
```

Salve como `soma.asm`

---

## Explicação Rápida

| Parte | O que faz |
|-------|-----------|
| `global _start` | Marca o ponto de entrada do programa |
| `section .text` | Seção onde vai o código |
| `_start:` | Rótulo que marca o início |
| `mov` | Copia valor para um registrador |
| `add` | Soma dois valores |
| `syscall` | Chama o sistema operacional |

---

## Registradores Principais

Pequenos espaços de memória **dentro da CPU**, muito rápidos:

- **rax**: acumula resultados, número da syscall
- **rdi**: 1º argumento (syscalls/funções)
- **rsi**: 2º argumento (syscalls/funções)
- **rcx, rdx, rbx, r8-r15**: de propósito geral

---

## Como Executar

```bash
# Monta (assembler)
nasm -f elf64 soma.asm -o soma.o

# Liga (linking)
ld soma.o -o soma

# Executa
./soma

# Ver resultado (código de saída)
echo $?
```

Deve imprimir **5**.

---

## Resumo dos Conceitos

- **Registradores**: memória rápida da CPU (rax, rdi, rsi...)
- **Instruções**: mov, add, syscall, sub...
- **Seções**: `.text` (código), `.data` (dados), `.bss` (não inicializados)
- **Rótulos**: marcam endereços (`_start:`)
- **Diretivas**: `global` torna símbolos visíveis
- **Syscalls**: pedem serviços ao kernel

---

## Equivalente em C

```c
int main() {
    int a = 2;
    int b = 3;
    return a + b;  // retorna 5
}
```

Em assembly, você faz isso manualmente.

---

## Exercícios

1. Mude o programa para somar 10 + 20
2. Implemente uma **subtração** (instrução `sub`)
3. Tente **multiplicação** (instrução `imul`, pesquise a sintaxe)
