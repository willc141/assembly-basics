# Assembly x86-64 - Fluxo de Controle

## Conceitos Essenciais

### RFLAGS (Registrador de Flags)
Armazena indicadores após operações:
- **ZF** (Zero Flag): 1 se resultado = 0
- **SF** (Sign Flag): 1 se resultado negativo
- **CF** (Carry Flag): 1 se houve estouro
- **OF** (Overflow Flag): 1 se transbordou

### CMP (Compare)
Subtrai dois valores, atualiza flags, descarta resultado.
```assembly
cmp rax, 5      ; rax - 5 (não modifica rax,. só flags)
```

### Saltos Condicionais (Jcc)
Desviam para um rótulo baseado em flags:

| Instrução | Condição | Exemplo |
|-----------|----------|---------|
| `je` | igual | `je .igual` |
| `jne` | diferente | `jne .diferente` |
| `jg` | maior (com sinal) | `jg .maior` |
| `jl` | menor (com sinal) | `jl .menor` |
| `jge` | maior ou igual | `jge .maior_igual` |
| `jle` | menor ou igual | `jle .menor_igual` |
| `ja` | acima (sem sinal) | `ja .acima` |
| `jb` | abaixo (sem sinal) | `jb .abaixo` |
| `jz` | zero (ZF=1) | `jz .zero` |
| `jnz` | não zero (ZF=0) | `jnz .nao_zero` |
| `jmp` | **incondicional** | `jmp .sempre` |

---

## Programa: Par ou Ímpar?

```assembly
global _start

section .data
    num         dq   7
    msg_par     db   "Par", 10
    tam_par     equ  $ - msg_par
    msg_impar   db   "Impar", 10
    tam_impar   equ  $ - msg_impar

section .text
_start:
    mov rax, [num]       ; carrega número
    test rax, 1          ; testa bit 0: par=0, ímpar=1
    jz .par              ; se zero (par), pula para .par

    ; Se ímpar
    mov rsi, msg_impar
    mov rdx, tam_impar
    jmp .escrever        ; pula a parte par

.par:
    mov rsi, msg_par
    mov rdx, tam_par

.escrever:
    mov rax, 1           ; syscall write
    mov rdi, 1
    syscall

    ; exit(0)
    mov rax, 60
    xor rdi, rdi         ; rdi = 0
    syscall
```

Salve como `par_impar.asm`

---

## Como Funciona

1. **Carrega número** em `rax`
2. **`test rax, 1`**: faz AND com 1 (testa bit menos significativo)
   - Se par (bit 0 = 0) → ZF = 1
   - Se ímpar (bit 0 = 1) → ZF = 0
3. **`jz .par`**: se ZF=1 (par), salta para `.par`
4. Se não saltou, executa código ímpar
5. **`jmp .escrever`**: pula a parte par
6. Em `.par`: carrega mensagem par
7. Em `.escrever`: chama write com a mensagem apropriada
8. Sai com exit(0)

---

## Fluxo

```
┌─ carrega num
│
├─ test: é par?
│  ├─ SIM (ZF=1) → salta para .par
│  └─ NÃO (ZF=0) → continua (ímpar)
│
├─ (ímpar) carrega msg_impar
│  └─ jmp .escrever
│
├─ .par: carrega msg_par
│
├─ .escrever: escreve (write)
│
└─ exit
```

---

## Executar

```bash
nasm -f elf64 par_impar.asm -o par_impar.o
ld par_impar.o -o par_impar
./par_impar
```

Com `num = 7` → `Impar`  
Com `num = 8` → `Par`

---

## Rótulos Locais

Rótulos com ponto (`.par`, `.escrever`) são **locais** e pertencem à última label global:
```assembly
_start:
    ...
    .par:      ; na verdade é _start.par
```

boa prática para evitar conflito de nomes

---

## Outros Usos Comuns

### Test
```assembly
test rax, 1      ; AND, descarta resultado, afeta flags
```

### Cmp
```assembly
cmp rax, 5       ; rax - 5, afeta flags
jg .maior        ; se rax > 5
```

### XOR (zera registrador)
```assembly
xor rdi, rdi     ; rdi = 0 (mais rápido que mov rdi, 0)
```

---

## Exercícios

1. **Modifique** para testar se número > 10. Imprima "Maior que 10" ou "Menor ou igual a 10".

2. **Loop 1-5** (desafio):
   - Use contador em registrador (ex: `rcx`)
   - Rótulo para início do loop
   - Incremente com `inc rcx`
   - Compare com `cmp rcx, 5` e `jle` para voltar
   - Para imprimir dígito: `add al, '0'` antes de escrever

3. **Desafio avançado**: Array de números, loop que soma todos:
   - Use loop com índice
   - Carregue com `mov rax, [array + rcx*8]`
   - Some com `add`
   - Converta resultado para string
