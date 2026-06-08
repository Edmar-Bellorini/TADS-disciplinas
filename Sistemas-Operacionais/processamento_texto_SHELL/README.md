# Processamento de Texto em SHELL (Bash)

> Pré-requisito: uso básico do SHELL (Bash)

---

## Sumário
1. [A arte de processamento de texto em SHELL](#1-a-arte-de-processamento-de-texto-em-shell)
2. [Comandos `wc` e `cut`](#2-comandos-wc-e-cut)
3. [Comandos `sort` e `uniq`](#3-comandos-sort-e-uniq)
4. [Usando PIPE `|` para conectar `sort` com `uniq`](#4-usando-pipe--para-conectar-sort-com-uniq)
5. [Comando `grep`](#5-comando-grep)
6. [Exercício](#6-exercício-com-wc-cut-sort-uniq-grep)
7. [Comando `sed`](#7-comando-sed)
8. [Exercício](#8-exercício-para-grep-e-sed)

---

## 1. A arte de processamento de texto em SHELL
A ideia de processamento de texto em Bash é reduzir problemas complexos em problemas menores que possam ser resolvidos por comandos especializados e conectá-los via `pipe` para gerar a solução completa. Os principais comandos são:

| Comando | Descrição |
|---------|-----------|
| `wc`    | conta palavras, linhas ou caracteres  |
| `cut`   | extrai campos ou colunas |
| `sort`  | ordena linhas |
| `uniq`  | remove linhas duplicadas  |
| `grep`  | filtra linhas com base em algum padrão |
| `sed`   | transformação básica em textos |
|~~`awk`~~   | ~~filtra, extrai, transforma textos (é uma linguagem de programação)~~

---

[voltar ao topo](#sumário)

---

## 2. Comandos `wc` e `cut`

### Comando `wc`

Apresenta o número de linhas, número de palavras e a quantidade de bytes de cada arquivo indicado

Sintaxe
```bash
wc [argumentos] arquivo_1 [arquivo_n]
```

Principais argumentos de `wc`
| Argumento | Descrição |
|:---------:|-----------|
| -c        | apresenta somente a quantidade de bytes     | 
| -m        | apresenta a quantidade de caracteres        | 
| -l        | apresenta o número de linhas                |
| -w        | apresenta o número de palavras              |

#### Exemplos `wc` com arquivo [`exemplo.sort.txt`](exemplo.sort.txt) 

> listar as informações de linhas, palavras e bytes do arquivo `exemplo.sort.txt`
```bash
wc exemplo.sort.txt
```

> listar somente o número de linhas do arquivo `exemplo.sort.txt`
```bash
wc -l exemplo.sort.txt
```

### Comando `cut`

Apresenta coluna(s) de cada arquivo indicado

Sintaxe
```bash
cut [argumentos] arquivo_1 [arquivo_n]
```
Principais argumentos de `wc`
| Argumento | Descrição |
|:---------:|-----------|
| -d`'s'`   | separador de campo `s`  (`TABULAÇÃO` utilizada em caso de não informado) |
| -f`x`     | campo `x` a ser apresentado                                              |
| -f`x[,y]` | campos `x` e `y` a serem apresentados                                    |
| -f`x-`    | campos de `x`até o final a serem apresentados                            |
| -s        | ignora linhas que não contém separador de campo                          |

#### Exemplos `cut` com arquivo [`exemplo.sort.txt`](exemplo.sort.txt) 

> Apresentar somente a lista de idades de `exemplo.sort.txt`
```bash
cut -d',' -f2 exemplo.sort.txt
```

> Apresentar todos os campos a partir de idade e ignorar linhas em branco do arquivo `exemplo.sort.txt`
```bash
cut -d',' -f2- -s exemplo.sort.txt
```

> Apresentar nome e salário, ignorando linha em branco do arquibo `exemplo.sort.txt`
```bash
cut -d',' -f1,3 -s exemplo.sort.txt
```

----
[voltar ao topo](#sumário)

----

## 3. Comandos `sort` e `uniq`

### Comando `sort`

Ordena as linhas de um arquivo, por padrão, em ordem crescente

Sintaxe
```bash 
sort [argumentos] arquivo
```

Principais argumentos de `sort`:
| Argumento | Descrição |
|:---------:|-----------|
| -n           | ordenação numérica                                 | 
| -r           | ordena em ordem descrescente                       | 
| -d           | ordena considerando apenas espaço em branco e alfanumérico |
| -u           | remove entradas duplicadas                         |
| -b           | ignora espaço em branco inicial para `-u`          |
| -t`s`        | indica o separador de colunas `s`                  |
| -k`x`        | ordena pela coluna `x`                             |
| -f           | sem distinção entre MAIÚSCULAS e minúsculas        |
| -o `arquivo` | escreve o resultado em `arquivo`                   |

#### Exemplos `sort` com arquivo [`exemplo.sort.txt`](exemplo.sort.txt) 

O arquivo [`exemplo.sort.txt`](exemplo.sort.txt) contém o `nome`, `idade` e `salário` de 30 personagens fictícios, e utiliza o separado de campo `,`.

>  Apenas ordenar em ordem crescente alfabética pelo nome
``` bash
sort exemplo.sort.txt
```
será apresentado na saída padrão a lista de nomes ordenado alfabeticamente. A ordenação ignora espaços em branco por padrão. 

Caso seja necessário considerar os espaços em branco, faz-se o uso da variável de ambiente `LC_ALL=C` antes do comando `sort` para não afetar a seção do emulador de terminal.

``` bash
LC_ALL=C sort exemplo.sort.txt
```

> Ordenar em ordem descrescente e considerando espaço em branco
``` bash
LC_ALL=C sort -r exemplo.sort.txt
```

> Ordenar em ordem descrescente, considerando espaço em branco e ignorando caracteres especiais
``` bash
LC_ALL=C sort -r -d exemplo.sort.txt
```
alternativamente:
``` bash
LC_ALL=C sort -rd exemplo.sort.txt
```

> Ordenar e remover duplicidade sem considerar espaço em branco
``` bash
sort -u exemplo.sort.txt
```

> Ordenar e remover duplicidade considerando espaço em branco (mantém primeira ocorrência antes da ordenação)
``` bash
sort -ub exemplo.sort.txt
```

> ordenar pela coluna da idade, porém, considerando os valores como texto

``` bash
sort -k2 -t, exemplo.sort.txt
```

> ordenar pela coluna da idade considerando os valores como número (correto)

``` bash
sort -k2 -t, -n exemplo.sort.txt
```

**Importante**: a primeira coluna é a de número 1, isto é, `-k1`

### Comando `uniq`

Lista linhas de um arquivo **ordenados** e omite duplicidade

> **Importante**: o comando `uniq` encontra duplicidade se as linhas estiverem de forma consecutivas, isto é, é necessário **ordená-las** primeiro.

Sintaxe
```bash 
uniq [argumentos] arquivo
```

Principais argumentos de `uniq`:
| Argumento | Descrição |
|:---------:|-----------|
| -c           | conta, remove e apresenta o número de duplicidades              | 
| -d           | somente retorna as duplicidades                                 | 
| -i           | sem distinção entre MAIÚSCULAS e minúsculas                     | 
| -u           | apresenta somente linhas únicas                                 |

#### Exemplos `sort` e `uniq` com arquivo [`exemplo.sort.txt`](exemplo.sort.txt) 

> Gerar o arquivo ordenado crescente pelo nome
```bash
sort -o exemplo.uniq.txt exemplo.sort.txt
```
> Lista somente as linhas dublicadas
```bash
uniq -d exemplo.uniq.txt
```
**importante**: mesmo ordenado, algumas linhas contém os mesmo dados, porém são diferentes, contendo um espaço em branco no início, que não é detectado como duplicidade pelo comando `uniq`.

----
[voltar ao topo](#sumário)

----

## 4. Usando PIPE `|` para conectar `sort` com `uniq`

Em Sistemas Operacionais Unix/Linux, existe o operador PIPE, dado pelo caractere `|`, que conecta a saída de um comando à entrada de outro comando

```
comando_1 --stdout--> | --sdtin--> comando_2
```

E pode ser utilizado indefinidamente

```
comando_1 --stdout--> | --sdtin--> comando_2 [--stdout--> | --sdtin--> comando_n]
```

A sintaxe, já visualizada acima é:
```bash
comando_1 | comando_2 [| comando_n]
```

#### Exemplo com `|`, `sort` e `uniq` com arquivo [`exemplo.sort.txt`](exemplo.sort.txt)

No exemplo anterior, foi criado o arquivo de saída `exemplo.uniq.txt` para poder usar o comando `uniq`, porém, se o arquivo de origem ocupar muito espaço, teremos 2 arquivos grandes, sendo o segundo, provavelmente, temporário. Desta forma, podemos usar o `|` para manter a saída do comando `sort` em memória e enviar como entrada ao `uniq`.

> Contar e listar somente as linhas duplicadas do arquivo `exemplo.sort.txt` ordenado decrescente pelo nome (em memória)

```bash
sort -r exemplo.sort.txt | uniq -dc
```

> Contar e listar somente o(s) nomes(s) das linhas duplicadas do arquivo `exemplo.sort.txt` ordenado decrescente pelo nome (em memória)

```bash
sort -r exemplo.sort.txt | uniq -dc | cut -d',' -f1
```

---

[voltar ao topo](#sumário)

---

## 5. Comando `grep`
Um dos comandos mais importantes do processamento de texto em SHELL. `grep` é utilizado para encontrar padrões de textos em linhas de um arquivo.

Sintaxe
```bash
grep [argumentos] "padrão a ser procurado" arquivo
```

Os principais argumentos são:

| Argumento | Descrição |
|:---------:|-----------|
| -i        | sem distinção entre MAIÚSCULAS e minúsculas                        | 
| -n        | mostra o número da linha juntamente com o resultado                |
| -l        | apresenta apenas o(s) nome(s) do(s) arquivo(s) que contém o padrão |
| -c        | conta quantas linhas contém o padrão pesquisado                    |
| -w        | procura a palavra inteira                                          |
| -x        | procura por linha inteira                                          |
| -E        | habilita expressões regulares      | 
| -r        | pesquisa em diretório (varios arquivos)                            |

No `padrão a ser procurado`, podemos utilizar algumas expressões de âncora, quantificadores e limitadores
| Argumento | Descrição | Exemplo  | Observação |
|:---------:|-----------|-----------|-----------|
| `^`     | âncora de início de linha | ^Erro | linhas que iniciam com "Erro"     |
| `$`     | âncora de fim de linha    | erro$ | linhas que terminam com "erro"    |
| `.`     | qualquer caractere        | c.t   | exemplos válidos: `cat` e `clt`   |
| `*`     | zero ou mais ocorrencias  | c.*t  | exemplos válidos: `ct`, `cat` e `count` |
| `[^x]`  | exceto o conjunto `x`     | [^8-9] | excluí digitos de `8` até `9`    |
| `\x`    | caractere especial `x`    | [\\.]   | linhas que contém `.`           |
| `\b`    | limitador de palavra      | [\bcat\b]   | linhas que contém `cat`     |

Podemos extender o comando `grep` com o argumento `-E` que permite uma melhor utilização dos agrupadores do tipo 
- `colchetes`: corresponde à qualquer um dos caracteres
- `parenteses`: corresponde à sequência de caracteres
Onde as principais alterações do `padrão a ser procurado` são:

| Argumento | Descrição | Exemplo  | Observação |
|:---------:|-----------|-----------|-----------|
| `[0-9]` | qualquer dígito           | [0-9]  | `0` até `9`                      |
| `[a-z]` | qualquer caractere minúsculo | [a-z]  | `a` até `z`                   |
| `[A-Z]` | qualquer caractere MAIÚSCULO | [A-Z]  | `A` até `Z`                   |
| `[a-Z]` | qualquer caractere        | [a-Z]  | `A` até `Z`                      |
| `(abc)` | somente a sequência `abc` | (abc)  | exemplos válidos: `aabcde` e `?abc_` |
| `(\(a\))`| somente a sequência `(abc)` | | a `\` foi usada para *escapar* os parênteses, isto é, `grep -E` entenderá os parênteses como caracteres |

e os modificadores quantitativos são: 

| Argumento | Descrição | Exemplo  | Observação |
|:---------:|-----------|-----------|-----------|
| `+`     | uma ou mais ocorrencias  | c+t  | exemplos válidos: `cat` e `cccat`      |
| `\|`    | operador `OR`  | (a\|b)   | linhas que contém `a` ou `b`                  |
| `{n}`   | `n` ocorrências do padrão | [0-9]{3} | válidos: 999, 365 e 000            |
| `{n,m}` | `n` até `m` ocorrências do padrão | [0-9]{2,4} | válidos: 99, 365 e 0001  |
| `{n,}`  | mínimo de `n` ocorrências do padrão | [0-9]{2,} | válidos: 99, 365,  0001 e 1365971  |
| `{0,m}` | máximo de `m` ocorrências do padrão | [0-9]{0,2} | válidos: 9 e 36        |

<!-- 123.123.456-00-->



#### Exemplos com `grep`

> Encontrar o nome `cloud` em algum arquivo do diretório atual
```bash
grep -ri "cloud" .
```

> Encontrar o nome `cloud` ou `yuna` em algum arquivo do diretório atual
```bash
grep -riE "(cloud|yuna)" .
```

> Encontrar o nome do processador da máquina em utilização 
```bash 
cat \proc\cpuinfo | grep "model name"
```

Será listado uma linha para cada processador virtual encontrado.

> Encontrar e mostrar uma única vez o nome do processador da máquina em utilização
```bash
cat \proc\cpuinfo | grep "model name" | uniq
```

> Encontar e listar o arquivo e o número da linha de padrão de CPF `999.999.999-99" no diretório atual
```bash
grep -rnE "[0-9]{3}.[0-9]{3}.[0-9]{3}-[0-9]{2}" .
```

> Mesmo sendo texto, é possível *tapacear* e encontrar linhas do arquivo `exemplo.sort.txt` com salários maiores ou iguais que `6000` 
```bash
grep -rE "(\b[0-9]*[6-9].{3}\b)" exemplo.sort.txt
```

## 6. Exercício com `wc`, `cut`, `sort`, `uniq`, `grep` 

O arquivo `IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao.csv` contém dados do censo de 2022 sobre o nível de instrução dos brasileiros. As colunas desta tabela são:

| Localidade | TOTAL sem instrução | TOTAL médio incompleto | TOTAL superior incompleto | TOTAL superior completo | HOMEM sem instrução | HOMEM médio incompleto | HOMEM superior incompleto | HOMEM superior completo | MULHER sem instrução | MULHER médio incompleto | MULHER superior incompleto | MULHER superior completo |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|

Considerações sobre o arquivo: 
- Este arquivo está compactado como `IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao.zip`
```bash
unzip IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao.zip
```
- Este arquivo não está ordenado alfabeticamente por `Localidade`

> Encontre:
> - Somente o valor de HOMENS `sem instrução` em `cascavel`
> - Somente o valor de MULHERES `superior incompleto` em `corbélia`

Utilize somente um ou mais comandos estudados até este momento:
- `wc`, `cut`, `sort`, `uniq`, `grep` 
- não esqueça do PIPE `|`
- são duas buscas, porém, uma dica para aproveitar e encontrar as linhas com `Cascavel` e `Corbélia` é:
```bash
grep -iE "(corb.lia|cascavel) \(PR\)" IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao.csv
```


---

[voltar ao topo](#sumário)

---

## 7. Comando `sed`

O comando `sed` é uma ferramenta para editar e processar texto diretamente no terminal ou em arquivos. Este opera com base em expressões regulares e pode realizar diversas tarefas, incluindo substituição de texto, exclusão de linhas e inserção de conteúdo.

Por padrão, o fluxo de execução do comando `sed` é:

1. Lê linha do arquivo
2. Executa o comando nesta linha
3. Vai para próxima linha
4. Repete 1.

Sintaxe
```bash
sed [argumentos] 'comando' arquivo
```

Os principais argumentos são:

| Argumento | Descrição |
|:---------:|-----------|
| `-n`      | Não imprime na saída padrão (modo silencioso) |
| `-z`      | Indica o caractere `NULL` como separador de linhas |
| `-E`      | Habilita o uso de expressões regulares (use aspas simples `'` com delimitador) |
| `-i`      | **ATENÇÃO:** Executa processamento no próprio arquivo (modo destrutivo) |
| `-i.bak`  | `-i`, porém, cria arquivo de backup `.bak` |

E as principais ações executadas em `comando` são:

| Ação | Descrição |
|:---------:|-----------| 
| `s/padrãobuscado/substituto/` | Substitui a **primeira** ocorrência de `padrãobuscado` por `substituto` |
| `/padrãobuscado/d`       | Deleta a linha que contém `padrãobuscado`  |
| `/padrãobuscado/i padrãoinserção` | Insere `padrãoinserção` antes das linhas com `padrãobuscado` |
| `/padrãobuscado/a padrãoinserção` | Insere `padrãoinserção` após as linhas com `padrãobuscado` |
| `comando/g`      | aplica `comando` em todas as ocorrências da linha |
| `comando/p`      | quando usado com argumento `-n`, imprime somente a linha que contém p `comando` | 


As âncoras `^`, `$`, `*` estudadas no comando `grep` também podem ser usadas com o comando `sed`. Também pode ser usado o caractere de escape `\`, por exemplo, para escapar espaços em branco (`\ `) 

Também existem os quantificadores, porém, diferentemente do comando `grep`, que faz uso de limitadores `{` e `}`, o comando `sed` requer a indicação antes das ações, podendo inclusive, substituir o `padrãobuscado` em alguns casos.

E para encerrar esta introdução sobre o comando `sed`, também é possível utilizar alguns recursos de expressão regular, como `[` e `]`, para encontrar um dos padrões, e `(` e `)`, para expressões um pouco mais complexas.

#### Exemplos com `sed`

> Inserir o nome `leonis` antes de `Noctis`
```bash
sed '/Noctis/i leonis' exemplo.sort.txt
```

O resultado será apresentado no próprio terminal, mantendo o arquivo original intacto (ação não destrutiva)

> Inserir o nome `leonis` antes de `Noctis` diretamente no arquivo `exemplo.sort.txt`, porém, criando um backup
```bash
sed -i.bak '/Noctis/i leonis' exemplo.sort.txt
```

**Importante:** o argumento `-i` é seguido de `.bak` como convenção. Por exemplo, o argumento `-i.backup` é válido.

> Substituir a nova entrada `leonis` por `Cor Leonis,45,12300` no arquivo exemplo

```bash
sed -i 's/leonis/Cor\ Leonis,45,12300/' exemplo.sort.txt
```

> Remover linhas em branco encontradas no arquivo `exemplo.sort.txt` sem alterá-lo (sem argumento `-i`)
```bash
sed '/^$/d' exemplo.sort.txt
```

> Imprimir somente a linha que ocorre a substituição (não destrutivo)
```bash
sed -n 's/Terra/Tera/p' exemplo.sort.txt
```

> Imprimir somente a décima linha do arquivo `exemplo.sort.txt` (não destrutivo)
```bash
sed -n '10p' exemplo.sort.txt
``` 

> Imprimir as linhas entre 6 e 9 do arquivo `exemplo.sort.txt` (não destrutivo)
```bash
sed -n '6,9p' exemplo.sort.txt
``` 

> Remover um espaço em branco no início das linhas do arquivo `exemplo.sort.txt` (não destrutivo)
```bash
sed 's/^ //' exemplo.sort.txt
```

> Remover linhas que contém somente espaço(s) em branco do arquivo `exemplo.sort.txt` (não destrutivo)
```bash
sed '/^ *$/d' exemplo.sort.txt
```
> Remover linhas que contém somente espaço(s) em branco ou Tabulação(ões) (não destrutivo)
```bash
sed '/^[ \t]*$/d' exemplo.sort.txt
```

> Remover os caracteres especiais dos nomes (não presenva acentuação) (não destrutivo)
```bash
sed 's/[^a-zA-Z,0-9 ]//g' exemplo.sort.txt
```
**Uma última explicação**
É tentador, neste momento, executar vários comandos `sed` utilizando `PIPE`para resolver vários problemas em um arquivo, por exemplo:

> Remover a linha em branco, a linha que contém espaços em branco, o espaço em branco no início do nome, os caracteres especiais, ordenar, remover duplicidade utilizando comandos `sed`, `sort` e `uniq`
```bash
sed '/^$/d' exemplo.sort.txt | sed '/^ *$/d' | sed 's/^ //' | sed 's/[^a-zA-Z,0-9 ]//g' | sort | uniq
```
Porém, o comando `sed` permite concatenar `comandos` utilizando o separador `;`. Para este exemplo, ainda precisamos usar o `PIPE` para ordenar com `sort` e remover duplicidade `duplicidade`
```bash
sed '/^$/d ; /^ *$/d ; s/^ // ; s/[^a-zA-Z,0-9 ]//g' exemplo.sort.txt | sort | uniq
```

## 8. Exercício para `grep` e `sed`

Utilizando novamente o arquivo `IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao.csv`, que contém a seguinte tabela:

| Localidade | TOTAL sem instrução | TOTAL médio incompleto | TOTAL superior incompleto | TOTAL superior completo | HOMEM sem instrução | HOMEM médio incompleto | HOMEM superior incompleto | HOMEM superior completo | MULHER sem instrução | MULHER médio incompleto | MULHER superior incompleto | MULHER superior completo |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|

Execute em uma única linha de comando do `SHELL`:
1. extraia todas as linhas que contém municípios do Parana, indicadas por `(PR)`
2. Remova a entrada `(PR)`
3. Ordene
4. Remova duplicidades
5. Grave no arquivo `IBGE_Censo_Demográfico_Tabela_2_Nivel_de_instrucao_PARANA.csv`



---

[voltar ao topo](#sumário)

---