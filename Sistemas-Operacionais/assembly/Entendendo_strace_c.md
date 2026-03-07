# Tentativa para entender o trace_c.txt

Ao executar o programa C com o `strace`, a saída revela dezenas de chamadas de sistema que ocorrem antes, durante e após a execução do código escrito pelo programador. Essas chamadas são geradas automaticamente pelo sistema operacional, pelo ligador dinâmico e pela `libc` e normalmente ficam completamente ocultas ao desenvolvedor.

A seguir, as principais syscalls agrupadas por finalidade.

---

## 1. Inicialização do processo
```
execve("./helloFile.x", ["./helloFile.x"], 0x7fff1a68a0b0 /* 46 vars */) = 0
```

O kernel recebe do shell a instrução para carregar o programa na memória. O `execve` é sempre a primeira syscall de qualquer processo, ele substitui o processo atual pelo novo programa, recebendo o caminho do executável, os argumentos e as variáveis de ambiente. Nenhuma linha do código C a invoca, é chamada pelo shell antes do programa iniciar.

---

## 2. Carregamento da libc
```
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=81503, ...}) = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF...", 832) = 832
pread64(3, ..., 784, 64) = 784
close(3)                                = 0
```

Antes de executar qualquer instrução do programa, o ligador dinâmico (`ld.so`) localiza e carrega a `libc` na memória. Para isso, consulta o arquivo `/etc/ld.so.cache` (um índice das bibliotecas disponíveis no sistema), abre o arquivo da `libc`, lê seu cabeçalho ELF para entender sua estrutura e o fecha após o mapeamento. O retorno `-1 ENOENT` na primeira linha indica que o arquivo `/etc/ld.so.preload` não existe, sendo o  comportamento normal na maioria dos sistemas.

---

## 3. Mapeamento de memória
```
mmap(NULL, 8192, PROT_READ|PROT_WRITE, ...)  = 0x7b8b93f98000
mmap(NULL, 2170256, PROT_READ, ...)          = 0x7b8b93c00000
mmap(0x7b8b93c28000, 1605632, PROT_READ|PROT_EXEC, ...) = 0x7b8b93c28000
mprotect(0x7b8b93dff000, 16384, PROT_READ)  = 0
munmap(0x7b8b93f84000, 81503)               = 0
brk(NULL)                                   = 0x5754b899a000
brk(0x5754b89bb000)                         = 0x5754b89bb000
```

O kernel mapeia a `libc` e outras regiões na memória virtual do processo. Cada chamada `mmap` reserva uma região com permissões específicas: leitura, escrita ou execução. O `mprotect` ajusta as permissões de regiões já mapeadas. O `munmap` libera regiões que não são mais necessárias. O `brk` expande o heap (pilha) do processo para acomodar as estruturas internas da `libc`.

---

## 4. Configuração do ambiente de execução
```
arch_prctl(ARCH_SET_FS, 0x7b8b93f81740) = 0
set_tid_address(0x7b8b93f81a10)         = 6562
set_robust_list(0x7b8b93f81a20, 24)     = 0
rseq(0x7b8b93f82060, 0x20, 0, 0x53053053) = 0
prlimit64(0, RLIMIT_STACK, NULL, {...})  = 0
getrandom("\x4a\x7e...", 8, GRND_NONBLOCK) = 8
```

A `libc` configura o ambiente de execução do processo antes de chamar o `main`. O `arch_prctl` configura o registrador `fs` para apontar para o armazenamento local da thread (`TLS`). O `set_tid_address` e `set_robust_list` preparam mecanismos de controle do thread. O `rseq` registra sequências reiniciáveis para operações atômicas. O `prlimit64` consulta os limites de recursos do processo, como o tamanho máximo da pilha. O `getrandom` obtém bytes aleatórios do kernel para inicializar mecanismos de segurança internos da `libc`.

---

## 5. O programa `helloWorldC` 
```
fstat(1, {st_mode=S_IFCHR|0620, ...})  = 0
write(1, "Hello from C!\n", 14)        = 14
exit_group(0)                          = ?
+++ exited with 0 +++
```

Somente aqui o código escrito pelo programador é executado. O `fstat` verifica as características do `stdout` antes de escrever. A `libc` utiliza essa informação para decidir a estratégia de buffer da escrita. O `write` realiza a escrita da mensagem no terminal. O `exit_group` encerra todos os threads do processo, diferente do `exit` direto utilizado no programa Assembly, sendo a forma que a `libc` utiliza para garantir um encerramento seguro em programas com múltiplos threads.

---

> **Nota Final:** das dezenas de syscalls exibidas pelo `strace`, apenas `write` e `exit_group` correspondem ao código escrito pelo programador. Todas as demais são geradas pela infraestrutura que o SO, o ligador dinâmico e a `libc` constroem automaticamente.

[Retornar à atividade](README.md)