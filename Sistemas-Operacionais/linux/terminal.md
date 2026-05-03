# Terminal

#### Sumário
- [Introdução](#1---introdução)
  - [Unix](#unix)
  - [Terminal](#terminal-unix)
  - [Terminal Contemporâneo](#terminal-contemporâneo)
- [Referências](#2---referências)


#### Objetivo


#### Requisitos
- Conhecimento básico sobre computadores e ter lido [Introdução a História do Linux](introducaoHistorica.md)

## 1 - Introdução

### Unix

Os computadores utilizados na década de 60 e 70 eram grandes e com custo elevado. O PDP-7 não era diferente, portanto, os pesquisadores que desenvolveram o [Unix](introducaoHistorica.md#unix) adicionaram suporte à **Multiusuário** e **Multitarefa**, possibilitando que vários usuários executassem seus programas "ao mesmo tempo" no mesmo computador.

O cenário era de que existia um único (ou poucos) computadores e vários usuários para utilizá-lo, e para resolver o problema de acesso, foi utilizado o conceito de **terminal**. 

### Terminal Unix

Terminal era um dispositivo físico com dois periféricos:
- Um teclado para enviar comando para o computador
- Um monitor de [tubo de raios catódicos](https://pt.wikipedia.org/wiki/Tubo_de_raios_cat%C3%B3dicos) para receber feedback

A imagem abaixo mostra uma abstração de como eram usados os computadores nesta época: vários terminais fisicamente distantes do computador e conectados por cabos a este.

<div style="text-align: center;">

<img src="./imagens/unix e terminal.svg" width="60%" height="60%">

*Reprodução: autor*
</div>

A forma de operar o computador principal:
1. inserir o comando pelo **teclado** do terminal
2.  pressionar `ENTER`, para enviar o comando ao **computador**
3. o computador, com sistema operacional **multiusuário** e **multitarefa**, que está localizado fisicamente distante do terminal, executa o comando
    1. o sistema operacional contém um programa chamado `Interpretador SHELL`, que recebe os comandos e chama outros programas correspondentes
    2. outros programas correspondentes comunicam-se com o kernel para executar as ações solicitadas
4. o computador retorna o resultado para o **terminal**
    1. seguindo a mesma sequência de camadas da execução
5. o terminal mostra para o usuário via **monitor**

> Observação: esta comunicação entre terminal e computador é apenas via troca de caracteres codificados em sinais elétricos.


### Terminal Contemporâneo

Atualmente em um computador pessoal, teclado, mouse e monitor fazem parte do mesmo "objeto" por assim dizer. Também levando em consideração os conceitos **Projeto GNU**, **Linux** e **Distribuição Linux**, tem-se um conjunto de camadas que o usuário utiliza para inserir, executar e obter respostas de comandos dados ao sistema computacional.

<div style="text-align: center;">

<img src="./imagens/camadas terminal01.svg" width="90%" height="90%">

*Reprodução: autor*
</div>

Os comandos inseridos pelo usuário utilizando o teclado são capturados pelo `EMULADOR DE TERMINAL`. Atualmente não usa-se o conceito de terminal separado do sistema computacional, porém, ainda é necessário para comunicação entre o usuário e o restante do sistema. Posteriomente, o `EMULADOR DE TERMINAL` envia os comandos para o `INTERPRETADOR SHELL`, que é o programa, pertencente ao **Projeto GNU**, que irá interpretar e executar as ferramentas correspondentes para execução da ação solicitada. As ferramentas em conjunto com o `INTERPETADOR SHELL`, executarão **chamadas de sistemas** do `KERNEL`, que comunicará com o `HARDWARE` do computador. 
A resposta do `HARDWARE` percorrerá o caminho inverso até alcançar o usuário através do `MONITOR`.

> Para simplicidade, os conceitos de `PSEUDOTERMINAL` **principal** e **secundário** não foram abordados, mas podem ser verificados no link [man-pages(pty)](https://man7.org/linux/man-pages/man7/pty.7.html).







|[← 1. Introdução a História do Linux](introducaoHistorica.md)|[↑ voltar ao topo](#terminal)|[⌂ página inicial](README.md)|[→ *3. todo*]()|

## 2 - REFERÊNCIAS

FREE SOFTWARE FOUNDATION. Philosophy of the GNU Project. [S.l.]: Free Software Foundation, [1996]. Disponível em: https://www.gnu.org/philosophy/philosophy.html. Acesso em: 19 de abril de 2026.

FREE SOFTWARE FOUNDATION. GNU/Linux FAQ. [S.l.]: Free Software Foundation, [1997]. Disponível em: https://www.gnu.org/gnu/gnu-linux-faq.html. Acesso em: 18 de abril de 2025.

HOLBROOK, Bernard D.; BROWN, W. Stanley. A history of computing research at Bell Laboratories (1937-1975). Murray Hill: Bell Telephone Laboratories, 1982. (Computing Science Technical Report, n. 99). Disponível em: https://archive.computerhistory.org/resources/access/text/2022/08/102804421-05-01-acc.pdf. Acesso em: 18 de abril de 2026.

KERRISK, Michael (ed.). pty(7): pseudoterminal interfaces. In: LINUX MAN-PAGES. Linux manual page, v. 6.16. [S.l.]: The Linux Kernel Organization, 17 maio 2025. Disponível em: https://man7.org/linux/man-pages/man7/pty.7.html. Acesso em: 2 de maio de 2026.

OPENSOURCE.COM. What is Bash? [S.l.]: Opensource.com, [s.d.]. Disponível em: https://opensource.com/resources/what-bash. Acesso em: 2 de maio de 2026.


