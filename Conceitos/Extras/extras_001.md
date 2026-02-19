Este resumo consolida tudo o que vimos até agora (até o 02) e adiciona conceitos importantes que ainda não foram explorados. Use como referência rápida e guia de boas práticas.

---

1. Estrutura de um programa assembly

Um programa assembly geralmente é dividido em seções:

```assembly
; Comentários começam com ponto e vírgula

global _start            ; torna o rótulo _start visível para o linker

section .data            ; dados inicializados (variáveis globais)
    <rótulo> <diretiva> <valores>

section .bss             ; dados não inicializados (reserva de espaço)
    <rótulo> <diretiva> <tamanho>

section .text            ; código executável
_start:                  ; ponto de entrada
    <instruções>
```

Diretivas comuns:

- db (define byte) – cada item ocupa 1 byte.
- dw (define word) – 2 bytes.
- dd (define double word) – 4 bytes.
- dq (define quad word) – 8 bytes.
- equ – define constante em tempo de montagem (ex: tamanho equ $ - mensagem).
- times – repete uma diretiva (ex: times 10 db 0).

Rótulos: São símbolos que representam endereços. Podem ser globais (sem ponto) ou locais (começam com ponto, ex: .loop). Rótulos locais são relativos ao último rótulo global.

---

2. Registradores

Na arquitetura x86-64, temos vários registradores de propósito geral e especiais.

Registradores de propósito geral (64 bits)

- RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, R8–R15

Cada um pode ser acessado em partes menores:

| 64 bits | 32 bits | 16 bits | 8 bits (alta) | 8 bits (baixa) |
|---------|---------|---------|---------------|----------------|
| `RAX` | `EAX` | `AX` | `AH` | `AL` |
| `RBX` | `EBX` | `BX` | `BH` | `BL` |
| `RCX` | `ECX` | `CX` | `CH` | `CL` |
| `RDX` | `EDX` | `DX` | `DH` | `DL` |
| `RSI` | `ESI` | `SI` | — | `SIL` |
| `RDI` | `EDI` | `DI` | — | `DIL` |
| `RBP` | `EBP` | `BP` | — | `BPL` |
| `RSP` | `ESP` | `SP` | — | `SPL` |
| `R8` | `R8D` | `R8W` | — | `R8B` |
Observações:

- Instruções que escrevem em registradores de 32 bits (ex: eax) zeram os 32 bits superiores do registrador de 64 bits correspondente.
- Instruções que escrevem em registradores de 16 ou 8 bits preservam os bits superiores (apenas a parte baixa é alterada).
- Os registradores RBP (base pointer) e RSP (stack pointer) têm funções especiais na pilha, mas podem ser usados para outros fins se necessário (com cuidado).

Registrador de flags (RFLAGS)

Não é acessível diretamente, mas influencia saltos condicionais. Principais flags:

- ZF (Zero Flag): setada se o resultado de uma operação for zero.
- SF (Sign Flag): setada se o resultado for negativo (msb=1).
- CF (Carry Flag): setada se houve "vai-um" em operações sem sinal.
- OF (Overflow Flag): setada se houve overflow em operações com sinal.
- PF (Parity Flag): paridade do byte baixo.

---

3. Instruções fundamentais

Movimentação de dados

- mov destino, fonte – copia dados. Ambos devem ter mesmo tamanho ou fonte imediata.
  - Ex: mov rax, 10 (imediato), mov rbx, rax (registrador), mov rcx, [mem] (memória → registrador), mov [mem], rdx (registrador → memória).
- movzx – move com extensão de zero (ex: movzx eax, byte [mem]).
- movsx – move com extensão de sinal (ex: movsx rax, word [mem]).
- xchg – troca dois operandos.

Aritmética inteira

| Instrução                 | Efeito                                                                                                                      |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `add destino, fonte`      | `destino += fonte`                                                                                                          |
| `sub destino, fonte`      | `destino -= fonte`                                                                                                          |
| `inc destino`             | `destino++`                                                                                                                 |
| `dec destino`             | `destino--`                                                                                                                 |
| `neg destino`             | `destino = -destino` (complemento de dois)                                                                                  |
| `mul fonte`               | Multiplicação sem sinal: `RAX * fonte` → resultado em `RDX:RAX`. Fonte pode ser registrador/memória de 8, 16, 32 ou 64 bits |
| `imul`                    | Multiplicação com sinal (várias formas: `imul reg, fonte`, `imul reg, fonte, imediato`, etc.)                               |
| `div fonte`               | Divisão sem sinal: divide `RDX:RAX` por `fonte`; quociente em `RAX`, resto em `RDX`                                         |
| `idiv`                    | Divisão com sinal                                                                                                           |
| `and`, `or`, `xor`, `not` | Operações lógicas bit a bit                                                                                                 |
| `shl`, `shr`              | Shift left/right lógico                                                                                                     |
| `sal`, `sar`              | Shift left/right aritmético (preserva sinal)                                                                                |
| `rol`, `ror`              | Rotate left/right                                                                                                           |

Comparação e testes

- cmp a, b – efetua a - b e atualiza flags (sem armazenar resultado).
- test a, b – efetua a AND b e atualiza flags (sem armazenar).

Saltos condicionais (baseados nas flags):

| Instrução     | Condição                   | Flags              |
| ------------- | -------------------------- | ------------------ |
| `jz` / `je`   | Igual / zero               | `ZF=1`             |
| `jnz` / `jne` | Diferente / não zero       | `ZF=0`             |
| `jg`          | Maior (signed)             | `ZF=0` e `SF=OF`   |
| `jge`         | Maior ou igual (signed)    | `SF=OF`            |
| `jl`          | Menor (signed)             | `SF != OF`         |
| `jle`         | Menor ou igual (signed)    | `ZF=1` ou `SF!=OF` |
| `ja`          | Acima (unsigned)           | `CF=0` e `ZF=0`    |
| `jae`         | Acima ou igual (unsigned)  | `CF=0`             |
| `jb`          | Abaixo (unsigned)          | `CF=1`             |
| `jbe`         | Abaixo ou igual (unsigned) | `CF=1` ou `ZF=1`   |
| `jc`          | Carry set                  | `CF=1`             |
| `jnc`         | Carry clear                | `CF=0`             |
| `jo`          | Overflow set               | `OF=1`             |
| `jno`         | Overflow clear             | `OF=0`             |
| `js`          | Sign set (negativo)        | `SF=1`             |
| `jns`         | Sign clear                 | `SF=0`             |

Saltos incondicionais

- jmp rótulo – desvia sempre.

Pilha e sub-rotinas

- push fonte – decrementa RSP e armazena fonte na pilha.
- pop destino – recupera valor do topo e incrementa RSP.
- call rótulo – empurra endereço de retorno e salta.
- ret – desempilha endereço e volta.

---

4. Acesso à memória

Em assembly, o acesso à memória é feito através de endereços. A sintaxe Intel usa colchetes [] para indicar "conteúdo da memória no endereço".

- Valor imediato: mov rax, 10 – coloca o número 10 em rax.
- Endereço de um rótulo: mov rsi, mensagem – coloca o endereço da mensagem em rsi.
- Conteúdo da memória: mov rax, [mensagem] – carrega 8 bytes a partir do endereço mensagem.
- Endereçamento indexado: mov rax, [rsi + rcx*8] – útil para arrays.
  - Formatos: [base + indice*escala + deslocamento] onde escala pode ser 1, 2, 4, 8.
- Endereçamento indireto: mov rax, [rbx] – carrega do endereço contido em rbx.

Cuidado: A diferença entre mov rsi, mensagem e mov rsi, [mensagem] é crucial. O primeiro coloca o endereço; o segundo, o conteúdo.

---

5. Chamadas de sistema no Linux (syscall)

A instrução syscall invoca o kernel. O número da chamada vai em RAX. Os argumentos (até 6) vão em RDI, RSI, RDX, R10, R8, R9 (nesta ordem). O valor de retorno fica em RAX (ou RDX:RAX se for 128 bits). Se erro, RAX recebe valor negativo.

• Syscalls comuns:

| Nome | Número (`rax`) | Argumentos |
|------|---------------|------------|
| `read` | 0 | `rdi=fd`, `rsi=buffer`, `rdx=count` |
| `write` | 1 | `rdi=fd`, `rsi=buffer`, `rdx=count` |
| `open` | 2 | `rdi=path`, `rsi=flags`, `rdx=mode` |
| `close` | 3 | `rdi=fd` |
| `exit` | 60 | `rdi=status` |

Exemplo:

```assembly
mov rax, 1          ; write
mov rdi, 1          ; stdout
mov rsi, mensagem   ; endereço
mov rdx, tamanho    ; número de bytes
syscall
```

---

### Boas práticas e dicas

1.  Comente bastante – assembly é críptico; comentários ajudam você e outros.
2. Use rótulos significativos – evite nomes genéricos como loop1.
3. Prefira registradores de 64 bits – a menos que precise de tamanhos menores.
4. Zere registradores com xor reg, reg – é mais eficiente que mov reg, 0.
5. Cuidado com div e idiv – eles usam RDX:RAX; sempre zere RDX antes se não quiser resto.
6. Salve registradores não voláteis – se você usar uma função que espera que certos registradores sejam preservados (convenção de chamada), você deve salvá-los na pilha. Mas no momento, estamos só no _start.
7. Use equ para tamanhos – evita erros de contagem manual.
8. Teste com valores pequenos – depure com gdb se algo der errado.
9. Entenda a diferença entre dados e código – não misture seções.
10. Prefira instruções curtas – por exemplo, inc é melhor que add reg, 1.

---

7. Erros comuns e como evitá-los

| Erro                           | Exemplo                                             | Consequência                                                 | Solução                                                |
| ------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| Esquecer colchetes             | `mov rax, mensagem` em vez de `mov rax, [mensagem]` | Carrega endereço, não conteúdo                               | Use colchetes para acessar memória                     |
| Usar colchetes para imediato   | `mov rax, [10]`                                     | Tenta acessar memória no endereço 10 (quase sempre inválido) | Não use colchetes para constantes                      |
| Tamanhos incompatíveis         | `mov al, [mensagem]` se mensagem for `dq`           | Carrega apenas 1 byte, possivelmente truncando               | Use registrador de mesmo tamanho dos dados             |
| Divisão sem zerar RDX          | `div rcx` com RDX sujo                              | Resultado incorreto ou overflow                              | `xor rdx, rdx` antes de `div`                          |
| Confundir `je` e `jz`          | Ambos são iguais                                    | —                                                            | Use `je` para igualdade, `jz` para zero; são sinônimos |
| Esquecer `global _start`       | —                                                   | Linker não acha ponto de entrada; erro de linking            | Sempre declare `global _start`                         |
| Usar syscall com número errado | `mov rax, 4` (antigo write)                         | Comportamento indefinido                                     | Consulte a tabela correta para 64 bits                 |


---

8. Tabela rápida de instruções aritméticas não vistas

| Instrução | Exemplo | Efeito |
|-----------|---------|--------|
| `adc` | `adc rax, rbx` | Soma com carry (`rax += rbx + CF`) |
| `sbb` | `sbb rax, rbx` | Subtração com borrow (`rax -= rbx + CF`) |
| `inc` | `inc rax` | `rax++` |
| `dec` | `dec rax` | `rax--` |
| `neg` | `neg rax` | `rax = -rax` |
| `mul` | `mul rbx` | `RDX:RAX = RAX * RBX` (unsigned) |
| `imul` | `imul rbx` | `RDX:RAX = RAX * RBX` (signed) |
| `div` | `div rbx` | Divide `RDX:RAX` por `RBX` (unsigned) |
| `idiv` | `idiv rbx` | Divide `RDX:RAX` por `RBX` (signed) |
| `and` | `and rax, rbx` | `rax &= rbx` |
| `or` | `or rax, rbx` | `rax \|= rbx` |
| `xor` | `xor rax, rbx` | `rax ^= rbx` |
| `not` | `not rax` | `rax = ~rax` |
| `shl` | `shl rax, cl` | `rax <<= cl` (shift left) |
| `shr` | `shr rax, cl` | `rax >>= cl` (lógico) |
| `sar` | `sar rax, cl` | Shift aritmético (preserva sinal) |
| `rol` | `rol rax, cl` | Rotate left |
| `ror` | `ror rax, cl` | Rotate right |

---

9. Exemplo com algns conceitos apresentados

```assembly
; programa que lê um número do usuário e imprime o dobro
; ilustrativo/

global _start

section .data
    prompt db "Digite um número (1 dígito): ", 0
    len_prompt equ $ - prompt
    msg_saida db "O dobro é: ", 0
    len_saida equ $ - msg_saida
    newline db 10

section .bss
    buffer resb 2        ; 1 char + newline

section .text
_start:
    ; escreve prompt
    mov rax, 1
    mov rdi, 1
    mov rsi, prompt
    mov rdx, len_prompt
    syscall

    ; lê 1 caractere
    mov rax, 0           ; read
    mov rdi, 0           ; stdin
    mov rsi, buffer
    mov rdx, 1           ; lê 1 byte
    syscall

    ; converte caractere para número (assume que é dígito)
    movzx rax, byte [buffer]
    sub rax, '0'
    ; dobra
    add rax, rax
    ; converte de volta para caractere
    add rax, '0'
    mov [buffer], al     ; guarda no mesmo buffer

    ; escreve mensagem de saída
    mov rax, 1
    mov rdi, 1
    mov rsi, msg_saida
    mov rdx, len_saida
    syscall

    ; escreve o dígito
    mov rax, 1
    mov rdi, 1
    mov rsi, buffer
    mov rdx, 1
    syscall

    ; newline
    mov rax, 1
    mov rsi, newline
    mov rdx, 1
    syscall

    ; exit
    mov rax, 60
    xor rdi, rdi
    syscall
```
