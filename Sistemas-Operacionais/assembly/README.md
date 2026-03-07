# `Hello World` em Assembly e Chamadas de Sistemas

#### Sumário
- [Preparação do Ambiente](#1---preparação-do-ambiente)
- [Hello World em C](#2---hello-world-em-c)
- [Compilando e Executando (código C)](#3---compilando-e-executando)
- [Executando com `strace` (código C)](#4---executando-com-strace)
- [Hello World em Assembly](#5---hello-world-em-assembly)
- [Montando, Ligando e Executando (código Assembly)](#6---montando-ligando-e-executando)
- [Executando com `strace` (código Assembly)](#7---executando-com-strace)
- [Atividade](#8---atividade-gravando-em-arquivo)

#### Objetivo

Introdução à linguagem de montagem Assembly e Chamadas de Sistemas utilizando sintaxe intel para arquiteturas [Intel® 64 and IA-32](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) e distribuições Linux.

#### Requisitos

- Ferramentas: `as`, `ld`, `gcc` e `strace`.
- Conhecimento básico em programação, CLI Linux e navegação de diretórios

## Introdução

Esta atividade é conduzida em etapas progressivas, iniciando por um programa **Hello World em linguagem C**, compilado com o `gcc` e executado com auxílio do `strace` para revelar as chamadas de sistema geradas
automaticamente pelo compilador e pela biblioteca padrão `libc`.

Na sequência, o mesmo programa é reescrito diretamente em **Assembly x86-64** com sintaxe Intel, eliminando qualquer camada intermediária e expondo como um programa se comunica com o Núcleo do Sistema Operacional, também chamado de **kernel** através das Chamadas de Sistemas.

## 1 - Preparação do Ambiente

Abra o terminal e navegue até o diretório de sua preferência, por exemplo:
```bash
cd ~/Documentos
```

Crie dois diretórios 'helloWorldC' e 'helloWorldA' para a atividade e acesse o diretório `helloWorldC':
```bash
mkdir helloWorldC
mkdir helloWorldA
cd helloWorldC
```

Crie o arquivo fonte:
```bash
touch helloWorldC.c
```

## 2 - Hello World em C

Abra o arquivo `helloWorldC.c` no editor de sua preferência e adicione o
seguinte código:
```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

### Entendendo o código

**`#include <stdio.h>`**  
Diretiva que inclui a biblioteca padrão de entrada e saída do C (`Standard Input/Output`). É ela que disponibiliza funções como `printf`. Sem essa linha, o compilador não reconheceria esta função e emitiria um `warning`.

**`int main()`**  
Ponto de entrada do programa: é a primeira função executada pelo sistema operacional. O tipo `int` indica que a função retorna um número inteiro ao Sistema Operacional ao encerrar.

**`printf("Hello, World!\n")`**  
Função da `stdio.h` que escreve texto na saída padrão (`stdout`). O `\n` é o caractere de nova linha, equivalente a pressionar Enter. Internamente, o `printf` realiza chamadas de sistema ao kernel para executar a escrita.

**`return 0`**  
Devolve o valor `0` ao Sistema Operacional, indicando que o programa encerrou com sucesso. Por convenção, qualquer valor diferente de `0` indica erro.

Salve o arquivo e retorne ao Terminal.

## 3 - Compilando e Executando

Compile o programa com o `gcc`:
```bash
gcc helloWorldC.c -o helloWorldC.x 
```

**Entendendo o comando:**

- `gcc helloWorldC.c`: invoca o compilador com o arquivo fonte
- `-o helloWorldC.x`: define o nome do arquivo executável gerado (".x" é sugestão de extensão)

Após a compilação, um arquivo executável `helloWorldC` será criado no
diretório atual. Execute-o:
```bash
./helloWorldC.x
```

A saída esperada é:
```
Hello, World!
```

## 4 - Executando com strace

Execute o programa novamente, desta vez sob o `strace`:
```bash
strace ./helloWorldC.x
```

A saída será extensa. Observe que o terminal exibirá dezenas de chamadas de sistema **antes** da mensagem `Hello, World!` aparecer, todas geradas automaticamente pelo compilador e pela `libc`, sem que o programador tenha escrito uma linha sequer sobre elas.

Redirecione a saída do `strace` para um arquivo para facilitar a leitura:
```bash
strace ./helloWorldC.x 2> trace_c.txt
cat trace_c.txt
```

Localize a chamada de sistema responsável pela escrita no terminal:
```
write(1, "Hello, World!\n", 14)         = 14
```

Observe também o encerramento do programa:
```
exit_group(0)                           = ?
+++ exited with 0 +++
```

> **Nota:** o `printf` da `libc` utiliza `exit_group` em vez de `exit`, uma syscall que encerra todos os threads do processo simultaneamente. Na versão Assembly, utilizaremos o `exit` diretamente.

Por fim, conte quantas syscalls foram geradas:
```bash
wc -l trace_c.txt
```

**Guarde esse número**, pois será comparado com a versão em Assembly.

## 5 - Hello World em Assembly

Retorne até o diretório 'helloWorldA'
Assembly:
```bash
cd  ../helloWorldA
```

Crie o arquivo fonte:
```bash
touch helloWorldA.s
```

Abra o arquivo `helloWorldA.s` no editor de sua preferência e adicione o seguinte código:
```asm
.intel_syntax noprefix
.section .data  # dados estaticos
    text:   .ascii "Ola do assembly\n"
    text_l = .-text
.section .text  # area de codigo
.global _start
_start:         # ponto de entrada
    # config da syscall WRITE
    # ssize_t write(int fd, const void buf[count], size_t count);
    # rax = write(rdi, rsi, rdx)
    mov rax, 1
    mov rdi, 1
    lea rsi, [rip+text]
    mov rdx, text_l
    syscall
    # config da syscall EXIT
    # void exit(int status);
    # void exit(rdi)
    mov rax, 60
    mov rdi, 0
    syscall
```

### Entendendo o código

**`.intel_syntax noprefix`**  
Diretiva que instrui o montador `as` a interpretar o código com a sintaxe Intel, dispensando os prefixos `%` (registradores) e `$` (imediatos). Sem ela, o `as` utilizaria a sintaxe AT&T, que é o padrão do GNU/Linux, porém menos intuitiva para iniciantes. Outra diferença entre as sintaxes é a ordem dos operandos `fonte, destino` (AT&T) ou `destino, fonte`(Intel). As seguintes linhas são códigos equivalente com sintaxe Intel (#a) e AT&T (#b)
```asm
    mov $60, %rax   #a
    mov rax, 60     #b
```

**`.section .data`**  
Define a seção de dados estáticos do programa, que armazena as variáveis e constantes conhecidas em tempo de montagem. Este trecho de código contém uma declaração de variável `text` (com valor conhecido) e a pseudoinstrução identificada por `text_l`.
```asm
text:   .ascii "Ola do assembly\n"
text_l = .-text
```

- `text`: rótulo que marca o endereço de início da string na memória
- `.ascii`: diretiva que reserva os bytes da string sem adicionar `\0` ao final
- `text_l = .-text`: calcula o tamanho da string: `.` representa o endereço atual e `.-text` resulta na diferença entre o endereço atual e o início da string. 

**`.section .text`**  
Define a seção de código executável do programa. 

> **Nota:** as seções `.section` podem aparecer em ordem diferente no código, porém, não podem ser fracionadas, isto é, caso o programador esqueça de uma constante, este deve escrevê-la dentro da seção `.data`, e não abrir outra seção de mesmo nome em outro local do código. Outras seções não utilizadas neste código são: `.rodata` (read-only), `.bss` (variáveis de valores não conhecidos), `.comment` (controle de versão manual/metadados) e `init`/`fini`(código para execução antes ou depois do ponto de entrada `_start`).

**`.global _start`**  
Torna o rótulo `_start` visível ao ligador `ld`. Sem essa diretiva, o ligador não  encontraria o ponto de entrada do programa. 

**`_start`**  
Ponto de entrada do programa que equivalente à função `main` em C. Neste momento, o Núcleo do Sistema Operacional transfere o controle da execução ao programa após carregá-lo em memória.

---

**Syscall `write`**
```asm
mov rax, 1          # número da syscall write
mov rdi, 1          # fd = 1 (stdout)
lea rsi, [rip+text] # endereço da string
mov rdx, text_l     # tamanho em bytes
syscall             # transfere controle ao kernel
```

O Núcleo do Sistema Operacional identifica a syscall pelo número em `rax` e recebe os argumentos em ordem pelos registradores `rdi`, `rsi` e `rdx`, esta chamada [`WRITE`](https://man7.org/linux/man-pages/man2/write.2.html) e sua equivalencia em assembly é:
```c
ssize_t write(int fd, const void buf[count], size_t count);
rax     write(rdi,    rsi,                   rdx)
```

A instrução **load effective adreess** `lea rsi, [rip+text]` utiliza endereçamento relativo ao registrador `rip`, que sempre aponta para a instrução atual, garantindo que o programa localize a string corretamente independente do endereço em que for carregado na memória.

> **Nota**: cada Chamada de Sistema tem um número associado que deve ser indicado no registrador `rax`. 

---

**Syscall `exit`**
```asm
mov rax, 60         # número da syscall exit
mov rdi, 0          # código de saída = 0 (sucesso)
syscall
```

Encerra o processo devolvendo o código `0` ao sistema operacional. Diferente do programa em `C` que utiliza `exit_group` via `libc`, aqui a chamada ao kernel é direta e explícita.

Salve o arquivo e volte ao terminal.

## 6 - Montando, Ligando e Executando

Ao contrário do programa em C, onde utilizamos apenas o comando `gcc` cobrindo todas as etapas de compilação, em Assembly o processo é realizado em duas etapas explícitas: **montagem** e **ligação**.

O `gcc` internamente executa essas mesmas etapas, porém de forma transparente ao programador. Aqui, cada etapa é executada manualmente.

**Etapa 1 — Montagem com `as`**  
O montador traduz o código Assembly em código de máquina, gerando
um arquivo objeto:
```bash
as helloWorldA.s -o helloWorldA.o
```

- `as`: invoca o montador
- `helloWorldA.s`: código fonte a ser montado
- `-o helloWorldA.o`: nome do arquivo do código **objeto**

> **Nota** arquivo **objeto** é o código Assembly traduzido para instruções de máquina, porém ainda não executável. Contém o código, os dados e uma tabela de símbolos com os rótulos do programa.

**Etapa 2 — Ligação com `ld`**  
O ligador combina o arquivo objeto com as dependências necessárias,
gerando o executável final:
```bash
ld helloWorldA-o helloWorldA.x
```

- `ld`: invoca o ligador
- `helloWorldA.o`: arquivo objeto a ser ligado
- `-o helloWorldA.x`: define o nome do executável gerado (`,x` é sugestão de extensão)

> **Nota:** diferente do programa em `C`, nenhuma biblioteca externa como a `libc` foi ligada e isso significa que o programa depende exclusivamente das syscalls do kernel.

**Execute o programa:**
```bash
./helloWorldA.x
```

A saída esperada é:
```
Ola do assembly
```

---

## 7 - Executando com strace

Execute o programa sob o `strace`:
```bash
strace ./helloWorldA.x
```

Redirecione a saída para um arquivo:
```bash
strace ./helloWorldA 2> trace_asm.txt
cat trace_asm.txt
```

A saída será semelhante a:
```
execve("./helloWorldA", ["./helloWorldA"], 0x... /* N vars */) = 0
write(1, "Ola do assembly\n", 16)       = 16
Ola do assembly
exit(0)                                 = ?
+++ exited with 0 +++
```

Observe que apenas **3 linhas** foram geradas: `execve`, `write` e `exit`.

Retorne um diretório acima e compare o número de syscalls entre os dois programas:
```bash
cd ../
wc -l helloWorldC/trace_c.txt
wc -l helloWorldA/trace_asm.txt
```

> **Nota:** todas as syscalls extras do programa C existem porque a `libc` precisa inicializar seu ambiente antes de executar qualquer linha de código, carregar configurações de locale, preparar o heap, registrar funções de encerramento, entre outras tarefas. O Assembly elimina toda essa camada e conversa diretamente com o kernel. [Quer saber um pouco mais sobre o conteúdo do arquivo `strace_c.txt`?](Entendendo_strace_c.md)

# 8 - Atividade Gravando em Arquivo

Crie um novo diretório para a atividade:
```bash
mkdir hello2File
cd hello2File
```

### Enunciado

Utilizando os conceitos apresentados nesta atividade, escreva um programa em Assembly x86-64 com sintaxe Intel no arquivo `hello2File.asm` que:

1. Crie um arquivo chamado `saida.txt` no diretório atual
2. Grave uma mensagem de sua escolha no arquivo
3. Feche o arquivo
4. Encerre o programa com código de saída `0`

### Requisitos

- Utilizar as syscalls `open`, `write`, `close` e `exit` diretamente
- Não utilizar nenhuma função de biblioteca externa
- Montar com `as` e ligar com `ld`
- Executar com `strace` e verificar se as quatro syscalls aparecem na saída
- O arquivo `saida.txt` deve ser aberto ou criado com a `flag de criação e somente para escrita (101o)` e com permissões `-rw-rw-r (0664o)`

#### Dicas

##### [Chamadas de Sistemas Linux em formato tabela por Paolo Stivanin](https://syscalls64.paolostivanin.com/)

##### Chamada de Sistema [Open](https://man7.org/linux/man-pages/man2/open.2.html) (mov rax, 2)
```c
int open(const char *path, int flags, mode_t mode );
rax open(             rdi,       rsi,          rdx)
```

> **Importante**: a Chamada de Sistema `Open` recebe o endereço de `path` (arquivo a ser aberto), e não recebe o tamanho deste texto, por este motivo, utilize `\0` após no nome do arquivo:
```asm
path: .ascii "saida.txt\0"
```

Para definir constantes em tempo de montagem, no início do código antes da `.section data`, entre com os valores para `flags`e `mode`:
```asm 
.set flags, 0101    # flag para criar + escrever
.set mode,  0664    # mode -rw-rw-r--
```

> **Nota**: o prefixo `0` indica que o número deve ser interpretado em base octal

e depois, utilizar a instrução `mov` para alocar o valor definido no regitrador específico
```asm
    mov rsi, flags  # flags
    mov rdx, mode   # mode
```


##### Chamada de Sistema [Close](https://man7.org/linux/man-pages/man2/close.2.html) (mov rax, 3)

> **Importante**: a Chamada de Sistema `open` retornará, em `rax` um número inteiro que representa o `filedescriptor (fd)`. 

> **Nota**: Registradores que podem ser usados: `rax, rbx, rdx, rcx, r8, r9, r10, r11, r12, r13, r14, r15`

```c
int close(int fd);
rax close(   rdi)
```

[voltar ao início](#sumário)