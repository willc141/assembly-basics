### O que é Assembly?

Assembly é uma linguagem de programação de baixo nível que corresponde quase diretamente às instruções que o processador entende (código de máquina). Cada instrução assembly representa uma operação simples, como mover dados, somar números ou desviar o fluxo. Diferente de C, onde você escreve expressões e estruturas de controle, em assembly você gerencia explicitamente os registradores, a memória e a pilha.

A arquitetura x86-64 é a usada na maioria dos computadores pessoais atuais (processadores Intel e AMD em modo 64 bits). Vamos usar a sintaxe Intel, que é mais legível (o destino fica à esquerda, a fonte à direita: mov destino, fonte). O ambiente será Linux, utilizando o assembler NASM e o linker ld.

### primeiro programa: somar dois números e sair

Vamos escrever um programa que soma dois números (2 e 3) e usa o resultado como código de saída do programa. No Linux, todo programa termina com uma chamada de sistema exit, que informa ao kernel que o processo deve ser encerrado, passando um número (código de retorno). Esse código pode ser capturado pelo shell com o comando echo $?

 Abaixo está o programa completo. Não se preocupe em entender por agora, veremos depois.

```assembly
; programa: soma.asm
; objetivo: soma 2 + 3 e termina com o resultado como código de saída

global _start               ; torna o símbolo _start visível para o linker

section .text               ; seção de código (instruções)

_start:                     ; ponto de entrada do programa
    mov rax, 60             ; número da syscall exit (60) -> colocado em rax
    mov rdi, 2              ; primeiro número da soma -> registrador rdi
    mov rsi, 3              ; segundo número da soma -> registrador rsi
    add rdi, rsi            ; rdi = rdi + rsi  (agora rdi vale 5)
    syscall                 ; chama o kernel para executar a syscall indicada em rax
```

Salve esse código em um arquivo chamado `soma.asm`

### Linha por Linha

1. `Comentários`

Tudo que vem após um ponto e vírgula (;) é um comentário – ignorado pelo assembler, serve apenas para nós, humanos.

2. `global _start`

Esta é uma diretiva do assembler, não uma instrução de máquina. Ela diz ao montador (NASM) que o rótulo `_start` deve ser visível globalmente, ou seja, o linker (ld) poderá enxergá-lo. O ponto de entrada de um programa em Linux (quando usamos ld diretamente) é o símbolo `_start`. Sem essa declaração, o linker não saberia onde o programa começa.

3. `section .text`

Todo programa em assembly é dividido em seções. A seção .text é onde colocamos o código executável (instruções). Outras seções comuns são .data (dados inicializados) e .bss (dados não inicializados), mas por enquanto só usaremos .text.

4. `_start:`

Um rótulo (label). É simplesmente um nome que marca um endereço de memória. O rótulo `_start` indica o endereço da primeira instrução do programa. O linker usará esse endereço como ponto de entrada.

5. `mov rax, 60`

A instrução mov (de move, ou mover em português) copia o valor da fonte para o destino. Aqui, rax é um registrador e 60 é um valor imediato (constante). Após esta instrução, o registrador rax conterá o número 60.

### O que é um registrador?

Registradores são pequenas áreas de memória dentro do processador, extremamente rápidas, usadas para armazenar dados temporários e endereços. Em x86-64, temos vários registradores de propósito geral, cada um com 64 bits (8 bytes) de tamanho. Os principais são: rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8 a r15. Cada um pode ser usado para qualquer operação aritmética ou movimentação, mas alguns têm funções especiais em certas instruções ou convenções.

rax é frequentemente usado para acumular resultados e também para armazenar o número da chamada de sistema (syscall) no Linux.

rdi e rsi são usados para passar argumentos para funções e syscalls (primeiro e segundo argumentos, respectivamente).

6. mov rdi, 2

Coloca o valor 2 no registrador rdi. rdi é um registrador de 64 bits, então, pode armazenar números até 2⁶⁴-1.

7. mov rsi, 3

Coloca o valor 3 no registrador rsi.

8. add rdi, rsi

A instrução add soma a fonte (rsi) ao destino (rdi). O resultado fica armazenado no destino (rdi). Após essa instrução, rdi conterá 5 (2 + 3). O registrador rsi permanece inalterado (ainda vale 3).

9. syscall

Essa é a instrução que faz uma chamada de sistema (system call). Ela transfere o controle para o kernel do Linux, que executa uma operação baseada no valor do registrador rax. No nosso caso, rax vale 60, que é o número da syscall exit (consulte a tabela de syscalls do Linux – cada SO tem seus números). A syscall exit espera um argumento: o código de saída, que deve estar em rdi. Portanto, o kernel encerra o processo e retorna ao shell o valor 5.

### Como executar esse programa

Precisamos transformar o código assembly em um executável. O processo consiste em:

1. Montagem (assembly): com o NASM, geramos um arquivo objeto (.o)
2. Ligação (linking): com o ld, unimos o código objeto a bibliotecas e geramos o executável

No terminal (Linux), faça:

```bash
nasm -f elf64 soma.asm -o soma.o
ld soma.o -o soma
```

· -f elf64 especifica o formato de saída (ELF64, comum em Linux 64 bits).
· O resultado é um binário chamado soma.

Agora execute:

```bash
./soma
```

Nada aparece na tela, pois o programa não escreve nada. Para ver o código de retorno, use:

```bash
echo $?
```

O shell deve imprimir 5. E então, você somou 2 e 3 em assembly.

### Conceitos importantes apresentados (resumo)

· Registradores: memórias internas da CPU, rápidas, com nomes como rax, rdi, rsi. São usados para operandos de instruções.
· Instruções: mov (cópia), add (soma), syscall (chamada de sistema).
· Seções: .text para código.
· Rótulos: `_start`: marca um endereço.
· Diretivas: global torna o rótulo visível externamente.
· Chamadas de sistema: mecanismo para pedir serviços ao kernel (como encerrar o programa, escrever algo, etc...)

### Comparação com C

Em C, o programa equivalente seria:

```c
int main() {
    int a = 2;
    int b = 3;
    int c = a + b;
    return c;
}
```

Quando compilado, o código de máquina gerado faria algo muito parecido com nosso assembly: usaria registradores para guardar a, b, c e faria uma instrução add. A diferença é que o compilador gerencia os detalhes, enquanto em assembly você controla tudo manualmente

### Exercícios

1. Modifique o programa para somar outros números (por exemplo, 10 e 20) e verifique o resultado com echo $?. 
2. Tente também fazer uma subtração (instrução sub – pesquise sobre a sintaxe)
