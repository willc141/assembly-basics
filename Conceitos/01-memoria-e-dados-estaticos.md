Anteriormente, aprendemos a usar registradores para fazer operações aritméticas e terminar o programa com exit. Agora vamos dar um passo além: armazenar dados na memória (variáveis) e usar uma chamada de sistema mais interessante, write, para escrever texto no terminal. Isso introduzirá os conceitos de seção .data, endereços de memória e a diferença entre valor imediato e acesso à memória.

### O que é a seção .data?

Até agora, todos os dados que usamos (números 2, 3, 50, 10) estavam embutidos nas próprias instruções como valores imediatos. Mas, para strings ou conjuntos de dados maiores, é mais prático armazená-los na memória principal (RAM) e acessá-los por endereços. A seção .data (dados) é onde declaramos variáveis estáticas inicializadas, ou seja, que já começam com um valor definido e permanecem na memória durante toda a execução

### Nosso programa: exibir "Olá, Assembly!"

Vamos escrever um programa que imprime a mensagem "Olá, Assembly!" na tela e depois termina com código 0 (sucesso). No Linux, a syscall para escrever em um arquivo (ou terminal) é a write, número 1. Ela exige três argumentos:

· rdi: file descriptor (1 = stdout, saída padrão)
· rsi: ponteiro para o buffer contendo os dados a escrever (endereço de memória onde a string está armazenada)
· rdx: número de bytes a escrever

Após a write, continuamos com exit (syscall 60) para encerrar o programa.

Observe que usamos a seção .data para guardar a string e calculamos automaticamente o seu tamanho..

```assembly
; programa: ola.asm
; objetivo: imprimir "Olá, Assembly!" no terminal

global _start               ; ponto de entrada visível para o linker

section .data               ; seção de dados inicializados
    mensagem db "Olá, Assembly!", 10   ; 10 é o código ASCII de newline ('\n')
    tamanho equ $ - mensagem            ; calcula o tamanho da string em tempo de montagem

section .text               ; seção de código

_start:
    ; chamada write(1, mensagem, tamanho)
    mov rax, 1              ; número da syscall write
    mov rdi, 1              ; file descriptor 1 = stdout
    mov rsi, mensagem       ; endereço da string (não o conteúdo)
    mov rdx, tamanho        ; número de bytes a escrever
    syscall                 ; executa a syscall

    ; chamada exit(0)
    mov rax, 60             ; número da syscall exit
    mov rdi, 0              ; código de retorno 0 (sucesso)
    syscall
```

### Linha por Linha

1. section .data

Aqui começamos a seção de dados. Tudo o que for declarado a seguir será colocado pelo montador na memória do programa, com permissões de leitura/escrita (e geralmente sem permissão de execução).

2. mensagem db "Olá, Assembly!", 10

Esta linha declara uma variável (ou melhor, um rótulo) chamada mensagem. O db significa define byte – ou seja, estamos definindo uma sequência de bytes com os valores fornecidos. A string "Olá, Assembly!" será convertida em seus códigos ASCII correspondentes. Depois dela, adicionamos uma vírgula e o número 10, que é o código ASCII do caractere newline (nova linha). Sem esse 10, a mensagem seria impressa e o cursor ficaria na mesma linha, o que pode não ficar tão agradável visualmente

Importante: Em assembly, as strings não terminam automaticamente com caractere nulo (0) como em C. Precisamos controlar explicitamente o tamanho ou usar um terminador se quisermos. Aqui optamos por calcular o tamanho explicitamente.

3. tamanho equ $ - mensagem

Esta é uma diretiva de montagem (equ), não uma instrução. Ela define um símbolo tamanho que será substituído pelo valor da expressão $ - mensagem durante a montagem.

· $ representa a posição atual (endereço) dentro da seção. Após a definição de mensagem, o montador continua; $ está apontando para o próximo byte após a string (incluindo o newline).
· mensagem é o endereço inicial da string.
· Portanto, $ - mensagem resulta no número de bytes ocupados pela string (incluindo o newline). Esse cálculo é feito em tempo de montagem, gerando uma constante.

Assim, não precisamos contar manualmente os caracteres; o montador faz isso por nós. É uma prática comum em assembly

4. section .text

Agora entramos na seção de código.

5. `_start`:

Rótulo de entrada do programa.

6. mov rax, 1 e mov rdi, 1

Configuramos o número da syscall (1 = write) e o primeiro argumento (file descriptor 1). Até aqui, é similar ao que já vimos.

7. mov rsi, mensagem

Esta instrução merece atenção. mensagem é um rótulo, que representa um endereço de memória (o endereço onde o primeiro byte da string está armazenado). Quando escrevemos mov rsi, mensagem (sem colchetes), estamos colocando esse endereço no registrador rsi. O processador não carrega o conteúdo da memória; ele apenas copia o valor do endereço (um número de 64 bits) para rsi. Isso é o que a syscall write espera: um ponteiro para o buffer.

Comparação com C: se tivéssemos char mensagem[] = "Olá...";, então mensagem em C é o endereço do primeiro elemento. O assembly mov rsi, mensagem equivale a rsi = mensagem (o ponteiro). Já mov rsi, [mensagem] (com colchetes) carregaria os primeiros 8 bytes da string em rsi – isso seria o conteúdo, não o endereço, e não serviria para write.

8. mov rdx, tamanho

Aqui tamanho é uma constante definida com equ. O montador substitui pelo valor calculado. Então rdx recebe o número de bytes a escrever.

9. syscall

Executa a syscall write. O kernel pegará os bytes a partir do endereço em rsi, lerá rdx bytes e os enviará para o stdout (descritor 1). Após a execução, o programa continua na próxima instrução.

10. As duas últimas instruções são a syscall exit com código 0, que já conhecemos.

### Montagem e execução

Os comandos são os mesmos da aula anterior:

```bash
nasm -f elf64 ola.asm -o ola.o
ld ola.o -o ola
./ola
```

### Conceitos novos apresentados

· Seção .data: local para armazenar dados inicializados.
· Diretiva db: define bytes na memória. Existem também dw (word = 2 bytes), dd (double word = 4 bytes), dq (quad word = 8 bytes) para outros tamanhos.
· Rótulos como endereços: um rótulo se traduz no endereço daquela posição de memória.
· Operador $: endereço atual durante a montagem.
· Cálculo de tamanho em tempo de montagem: tamanho equ $ - mensagem.
· Acesso à memória: a distinção entre mov reg, rotulo (endereço) e mov reg, [rotulo] (conteúdo). Por enquanto, só usamos o primeiro, mas é crucial entender a diferença.
· Syscall write: necessita do endereço do buffer e do tamanho.

### Comparação com C

Em C, o programa equivalente seria:

```c
#include <unistd.h>

int main() {
    char mensagem[] = "Olá, Assembly!\n";
    write(1, mensagem, sizeof(mensagem)-1); // -1 se não quiser incluir o \0
    return 0;
}
```

Observe que mensagem em C também é um ponteiro (decai para ponteiro) quando passado para write. A diferença é que em C a string teria um terminador nulo implícito se usada como string literal, e sizeof inclui o terminador. Em assembly, não há terminador automático; controlamos o tamanho explicitamente.

### Exercícios

1. Modifique a mensagem para imprimir seu nome. Lembre-se de ajustar o db e o cálculo do tamanho.
2. Em vez de colocar o newline (10) dentro da string, remova-o e adicione uma segunda chamada write para imprimir apenas um newline. Dica: defina uma variável nova_linha db 10 e calcule seu tamanho (que será 1). Faça duas syscalls seguidas.
3. (Desafio) Crie um programa que soma dois números (como na aula 1), mas agora armazene esses números na memória (em .data) e carregue-os para os registradores antes de somar. Use mov reg, [num1] para carregar o valor. Não esqueça de definir os números com dq (quad word) para caberem em 64 bits. Ao final, armazene os números na memória, some e guarde o resultado de volta na memória (use mov [resultado], rax). Depois, faça um exit com o resultado.