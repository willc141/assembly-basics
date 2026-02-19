Anteriormente, vimos movimentação de dados, operações aritméticas, acesso à memória e chamadas de sistema. Agora vamos dar um passo fundamental: controlar o fluxo do programa, ou seja, fazer o computador tomar decisões (if/else) e repetir blocos de código (loops). Para isso, usaremos instruções de comparação e saltos condicionais.

Até agora, nosso programa executava instruções sequencialmente, uma após a outra, do início ao fim. Com saltos, podemos pular para outras partes do código ou voltar, dependendo de condições.

Conceitos básicos

· EFLAGS / RFLAGS: É um registrador especial (de 64 bits, mas apenas alguns bits são usados) que armazena flags (indicadores) resultantes de operações aritméticas e lógicas. Por exemplo, após uma instrução add ou sub, flags como ZF (zero flag), SF (sign flag), CF (carry flag), OF (overflow flag) são atualizadas automaticamente.

· CMP (compare): Instrução que subtrai o segundo operando do primeiro, mas descarta o resultado, apenas atualizando as flags. É como sub sem armazenar o resultado. Serve para comparar dois valores.

· Saltos condicionais (Jcc): Instruções que desviam o fluxo para um rótulo se determinada condição (baseada nas flags) for verdadeira. 

Exemplos: 
je (pule se igual)
jne (pule se diferente)
jg (pule se maior)
jl (pule se menor)
jge (pule se maior ou igual)
jle (pule se menor ou igual)
e etc.
Existem também saltos incondicionais: jmp (sempre pula).

### Decidir se um número é par ou ímpar

Vamos escrever um programa que verifica se o número armazenado em num (na memória) é par ou ímpar. Dependendo do resultado, ele imprimirá uma mensagem diferente. Para simplificar, em vez de imprimir strings completas, usaremos a saída da syscall write com mensagens curtas. Mas, para não complicar com duas mensagens diferentes, faremos algo mais simples: se o número for par, o programa retorna código 0 e imprime o número e uma mensagem indicando se é par ou impar. Para isso, precisaremos de duas strings diferentes. Também usaremos uma comparação para decidir qual imprimir

Primeiro, o programa em C equivalente seria algo como:

```c
int num = 7;
if (num % 2 == 0)
    write(1, "Par\n", 4);
else
    write(1, "Impar\n", 6);
return 0;
```

Em assembly, precisamos:

· Armazenar o número em .data.
· Calcular num % 2. Como é divisão por 2, basta testar o bit menos significativo (LSB). Se for 0, é par; se for 1, é ímpar. Isso pode ser feito com test ou and.
· Usar um salto condicional para escolher a mensagem correta.
· Chamar write com a mensagem apropriada.
· Sair.

Vamos usar a instrução test para verificar o bit 0. test rax, 1 faz um AND bit a bit entre rax e 1, setando as flags ZF (zero) se o resultado for zero (ou seja, se o bit 0 for 0). Portanto, se o número for par, test resulta em zero (ZF=1); se for ímpar, resulta em não-zero (ZF=0). Então podemos usar jz (jump if zero) para pular se for par, ou jnz (jump if not zero) para pular se for ímpar.

### Código assembly

```assembly
; programa: par_impar.asm
; objetivo: verifica se um número é par ou ímpar e imprime mensagem correspondente

global _start

section .data
    num         dq   7          ; número a ser testado
    msg_par     db   "Par", 10   ; "Par" + newline
    tam_par     equ  $ - msg_par
    msg_impar   db   "Impar", 10 ; "Impar" + newline
    tam_impar   equ  $ - msg_impar

section .text
_start:
    ; Carregar o número para um registrador
    mov rax, [num]       ; rax = num

    ; Testar o bit menos significativo (LSB)
    test rax, 1          ; rax AND 1 (afeta flags, não modifica rax)
    jz   .par            ; se ZF=1 (resultado zero) -> é par, pula para .par

    ; Se chegou aqui, é ímpar
    mov rsi, msg_impar
    mov rdx, tam_impar
    jmp .escrever        ; pula a parte do par

.par:
    mov rsi, msg_par
    mov rdx, tam_par
    ; continua para .escrever

.escrever:
    ; Chamada write(1, rsi, rdx)
    mov rax, 1
    mov rdi, 1
    syscall

    ; Exit(0)
    mov rax, 60
    xor rdi, rdi         ; zera rdi (equivale a mov rdi, 0, mas mais eficiente)
    syscall
```

# Linha por Linha

1. A instrução test faz um AND bit a bit entre os dois operandos e descarta o resultado, apenas setando as flags de acordo. É equivalente a and rax, 1 mas sem modificar rax. As flags mais importantes aqui:

· ZF (Zero Flag) = 1 se o resultado for zero.
· SF (Sign Flag) = 1 se o resultado for negativo (mas como é AND, o resultado nunca é negativo porque 1 é positivo, então SF=0).
· Outras flags (CF, OF) são zeradas.

Se o bit 0 de rax for 0, o resultado de rax AND 1 é 0, então ZF=1. Se for 1, o resultado é 1, ZF=0.

2. jz .par

jz significa "jump if zero". Se a flag ZF estiver setada (1), o processador desvia a execução para o rótulo .par. Caso contrário, continua na próxima instrução (que é a sequência para ímpar).

3. Rótulos locais (.par, .escrever)

Observe os rótulos com um ponto no início: .par e .escrever. Em NASM, esses são rótulos locais, que são relativos ao último rótulo global/não-local. Neste caso, o último rótulo global é `_start`. Portanto, .par é na verdade `_start.par`. Isso ajuda a organizar o código sem poluir o espaço de nomes global. É uma boa prática usar rótulos locais para partes internas de uma função.

4. jmp .escrever

jmp é um salto incondicional. Após configurar os registradores para a mensagem ímpar, precisamos pular a parte que configuraria a mensagem par, caso contrário, cairíamos nela também. O jmp desvia para o rótulo .escrever, onde a syscall é feita.

5. .par: e continuação

No rótulo .par, configuramos rsi e rdx para a mensagem "Par". Depois disso, não há um salto, então a execução continua na próxima instrução, que é .escrever:. Isso é proposital: depois de configurar a mensagem par, queremos ir para a parte comum de escrita.

6. xor rdi, rdi

Esta instrução faz um XOR entre rdi e ele mesmo, resultando em zero. É uma forma comum e eficiente de zerar um registrador (equivale a mov rdi, 0, mas geralmente é mais rápido e ocupa menos bytes). xor também afeta flags, mas não nos importamos aqui.

Fluxo do programa

1. Carrega num em rax.
2. Testa se é par.
3. Se for par, pula para .par; senão, segue para a parte ímpar.
4. Na parte ímpar: carrega endereço e tamanho da mensagem ímpar e salta para .escrever.
5. Na parte par: carrega endereço e tamanho da mensagem par e cai em .escrever (sem salto).
6. Em .escrever: chama write com os parâmetros já configurados.
7. Sai.

### Montagem e execução

```bash
nasm -f elf64 par_impar.asm -o par_impar.o
ld par_impar.o -o par_impar
./par_impar
```

Deverá imprimir "Impar" (pois 7 é ímpar). Se mudar num para 8, imprimirá "Par".

### Outras formas de comparação

Além de test, podemos usar cmp para comparações numéricas gerais. Por exemplo, para verificar se rax é maior que 5:

```assembly
cmp rax, 5
jg  .maior       ; pula se rax > 5 (signed)
ja  .acima       ; pula se rax > 5 (unsigned)
```

jg (jump if greater) considera números com sinal (positivos/negativos). ja (jump if above) considera números sem sinal. Similarmente, jl (less) e jb (below) para menor.

### Exercícios

 • Desafios são opcionais, mas recomendado; se não conseguir resolver, não se preocupe, eles abordam conceitos mais avançados que geralmente não chegamos.

1. Modifique o programa para testar se o número é maior que 10. Se for, imprima "Maior que 10", senão imprima "Menor ou igual a 10". Use cmp e jg/jle.
2. Desafio 1: Crie um loop que imprima os números de 1 a 5, cada um em uma linha. Dica: use um registrador como contador, um rótulo para o início do loop, e incremente o contador até um limite. Use cmp e jl para continuar. (imprima apenas o caractere correspondente ao dígito, somando '0' ao número. Exemplo: para imprimir '1', faça add al, '0' e depois mov [buffer], al e chame write com um buffer de 1 byte.)
3. Desafio 2: Combine a aula 2 e 3: tenha um array de números (em .data) e percorra-o em loop, somando todos os elementos. No final, imprima a soma (em decimal) – isso exigirá conversão de número para string, que é um pouco mais complexo.
