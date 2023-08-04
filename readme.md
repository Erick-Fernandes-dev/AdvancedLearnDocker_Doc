# Day-1 


## Descomplicando Namespace

### O que é namespace?

Namespace é um conceito que permite que você tenha um escopo separado para cada conjunto de nomes. Isso significa que um nome de classe tem o mesmo significado em diferentes namespaces. Um namespace é projetado para superar o problema de conflito de nome. Por exemplo, você pode ter uma classe chamada MyClass em um namespace e a mesma classe MyClass em outro namespace. Essas duas classes são totalmente independentes. O namespace é uma coleção de classes.


## amespaces

Namespaces foram adicionados no kernel Linux na versão 2.6.24 e são eles que permitem o isolamento de processos quando estamos utilizando o Docker. São os responsáveis por fazer com que cada container possua seu próprio environment, ou seja, cada container terá a sua árvore de processos, pontos de montagens, etc., fazendo com que um container não interfira na execução de outro. Vamos saber um pouco mais sobre alguns namespaces utilizados pelo Docker.

## PID namespace

O PID namespace permite que cada container tenha seus próprios identificadores de processos. Isso faz com que o container possua um PID para um processo em execução -- e quando você procurar por esse processo na máquina host o encontrará; porém, com outra identificação, ou seja, com outro PID.

A seguir temos o processo "testando.sh" sendo executado no container.

Perceba o PID desse processo na árvore de processos dele:
```shell
root@c774fa1d6083:/# bash testando.sh &
[1] 7

root@c774fa1d6083:/# ps -ef
UID  PID PPID C STIME TTY TIME     CMD
root 1   0    0 18:06 ?   00:00:00 /bin/bash
root 7   1    0 18:07 ?   00:00:00 bash testando.sh
root 8   7    0 18:07 ?   00:00:00 sleep 60
root 9   1    0 18:07 ?   00:00:00 ps -ef

root@c774fa1d6083:/#

```

Agora, perceba o PID do mesmo processo exibido através do host:
```shell
root@linuxtips:~# ps -ef | grep testando.sh

root 2958 2593 0 18:12 pts/2 00:00:00 bash testando.sh
root 2969 2533 0 18:12 pts/0 00:00:00 grep --color=auto testando.sh

root@linuxtips:~#

```

Diferentes, né? Porém, são o mesmo processo. :)

## Net namespace

O Net Namespace permite que cada container possua sua interface de rede e portas. Para que seja possível a comunicação entre os containers, é necessário criar dois Net Namespaces diferentes, um responsável pela interface do container (normalmente utilizamos o mesmo nome das interfaces convencionais do Linux, por exemplo, a eth0) e outro responsável por uma interface do host, normalmente chamada de veth* (veth + um identificador aleatório). Essas duas interfaces estão linkadas através da bridge Docker0 no host, que permite a comunicação entre os containers através de roteamento de pacotes.

Conforme falamos, veja as interfaces. Interfaces do host:
```shell
root@linuxtips:~# ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:1c:42:c7:bd:d8 brd ff:ff:ff:ff:ff:ff
        inet 10.211.55.35/24 brd 10.211.55.255 scope global eth1
            valid_lft forever preferred_lft forever
        inet6 fdb2:2c26:f4e4:0:21c:42ff:fec7:bdd8/64 scope global dynamic
            valid_lft 2591419sec preferred_lft 604219sec
        inet6 fe80::21c:42ff:fec7:bdd8/64 scope link
            valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:c7:c1:37:14 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 scope global docker0
            valid_lft forever preferred_lft forever
        inet6 fe80::42:c7ff:fec1:3714/64 scope link
            valid_lft forever preferred_lft forever
5: vetha2e1681: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
        link/ether 52:99:bc:ab:62:5e brd ff:ff:ff:ff:ff:ff
        inet6 fe80::5099:bcff:feab:625e/64 scope link
             valid_lft forever preferred_lft forever
root@linuxtips:~#
```

Interfaces do **container**:

```shell
root@6ec75484a5df:/# ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
            valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
            valid_lft forever preferred_lft forever
6: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.3/16 scope global eth0
            valid_lft forever preferred_lft forever
        inet6 fe80::42:acff:fe11:3/64 scope link
            valid_lft forever preferred_lft forever

root@6ec75484a5df:/#
```
Conseguiu visualizar as interfaces Docker0 e veth* do host? E a eth0 do container? Sim? Otémooo! :D

## Mnt namespace

É evolução do chroot. Com o Mnt Namespace cada container pode ser dono de seu ponto de montagem, bem como de seu sistema de arquivos raiz. Ele garante que um processo rodando em um sistema de arquivos não consiga acessar outro sistema de arquivos montado por outro Mnt Namespace.

## IPC namespace

Ele provê um SystemV IPC isolado, além de uma fila de mensagens POSIX própria.
## UTS namespace

Responsável por prover o isolamento de hostname, nome de domínio, versão do SO, etc.

## User namespace

O mais recente namespace adicionado no kernel Linux, disponível desde a versão 3.8. É o responsável por manter o mapa de identificação de usuários em cada container.


### Como funciona?

O namespace é uma coleção de classes. O namespace é usado

```shell
debootstrap stable /debian http://deb.debian.org/debian 
```

O comando que você forneceu é um exemplo de uso do utilitário `debootstrap`. O `debootstrap` é uma ferramenta utilizada em sistemas Linux para criar um ambiente básico de uma distribuição Debian em um diretório específico, permitindo a criação de chroots ou até mesmo instalações completas de sistemas Debian em diretórios separados.

Vamos analisar os elementos do comando que você forneceu:

1. `debootstrap`: Este é o nome do utilitário que você está executando. Ele é usado para criar um ambiente Debian mínimo em um diretório de destino.

2. `stable`: Isso especifica a versão (ou codinome) do Debian que você deseja instalar. No caso do "stable", refere-se à versão estável atual do Debian.

3. `/debian`: Este é o diretório de destino onde o ambiente Debian será criado. Você pode substituir "/debian" por qualquer caminho de diretório de sua escolha.

4. `http://deb.debian.org/debian`: Este é o URI (Uniform Resource Identifier) do repositório Debian que será usado para baixar os pacotes necessários durante o processo de instalação. O `debootstrap` irá buscar pacotes a partir deste repositório para criar o ambiente Debian.

Em resumo, o comando que você forneceu está instruindo o `debootstrap` a criar um ambiente Debian estável (stable) no diretório "/debian", usando o repositório "http://deb.debian.org/debian" para baixar os pacotes necessários.

Esse tipo de comando é útil em várias situações, como a criação de ambientes de desenvolvimento isolados, a recuperação de sistemas quebrados ou a realização de tarefas de manutenção em sistemas Debian.

```shell

unshare --mount --uts --ipc --net --map-root-user --user --pid --fork chrrot /debian bash
```

A linha de comando que você forneceu está usando o comando `unshare` em conjunto com o utilitário `chroot` para criar um ambiente isolado (chroot jail) em um diretório específico e, em seguida, executar um shell bash dentro desse ambiente isolado. Vamos analisar os elementos do comando:

1. `unshare`: O comando `unshare` é usado para criar um novo ambiente de namespaces em Linux. Os namespaces são mecanismos de isolamento que permitem criar ambientes separados para processos, sistemas de arquivos, nomes de host, IPC (Inter-Process Communication) e rede. No comando fornecido, vários tipos de namespaces estão sendo criados.

2. `--mount`, `--uts`, `--ipc`, `--net`, `--map-root-user`, `--user`, `--pid`: Essas opções do `unshare` especificam quais namespaces devem ser criados e isolados. Cada uma delas controla um aspecto diferente do ambiente de isolamento, como montagens de sistema de arquivos, identificação do sistema, comunicação entre processos, rede, controle de usuário, IDs de processo e outros.

3. `chroot /debian`: O comando `chroot` é usado para alterar o diretório raiz do processo atual para um diretório específico. No caso, estamos alterando o diretório raiz para "/debian", que deve ser o mesmo diretório que você criou anteriormente com o `debootstrap`.

4. `bash`: Isso é o que será executado após criar o ambiente isolado. Neste caso, um shell bash será iniciado dentro do ambiente chroot isolado, permitindo que você interaja com o sistema de arquivos e execute comandos dentro desse ambiente.

Resumindo, essa linha de comando cria um ambiente isolado usando diferentes namespaces através do `unshare`, define o diretório raiz desse ambiente isolado usando o `chroot` e, em seguida, inicia um shell bash dentro desse ambiente isolado, permitindo que você execute comandos como se estivesse dentro de um sistema separado dentro do diretório "/debian". Isso pode ser útil para tarefas de depuração, testes ou recuperação em um ambiente controlado e isolado.

```shell

mount -t proc none /proc

```

O comando que você forneceu está montando o sistema de arquivos proc no diretório "/proc". O sistema de arquivos proc é um sistema de arquivos virtual que fornece informações sobre processos e outros recursos do sistema. Ele é usado principalmente para obter informações sobre processos em execução no sistema, como IDs de processo, uso de memória, uso de CPU, etc.

O comando que você forneceu é usado para montar o sistema de arquivos "proc" no diretório "/proc". Vamos entender o que cada parte do comando faz:

1. `mount`: Este é o comando utilizado para montar sistemas de arquivos em diretórios específicos.

2. `-t proc`: A opção `-t` é usada para especificar o tipo de sistema de arquivos a ser montado. Neste caso, está sendo especificado o tipo "proc", que é um sistema de arquivos virtual usado pelo kernel Linux para expor informações sobre processos e outros detalhes do sistema.

3. `none`: Isso indica que nenhuma fonte de dados específica está sendo usada para montar o sistema de arquivos. O "none" é comumente usado para sistemas de arquivos virtuais.

4. `/proc`: Este é o diretório de destino onde o sistema de arquivos "proc" será montado. O sistema de arquivos "proc" é uma maneira de interagir com o kernel e obter informações sobre os processos em execução, consumo de recursos do sistema e outros detalhes.

Portanto, o comando `mount -t proc none /proc` está montando o sistema de arquivos "proc" no diretório "/proc", permitindo que você acesse informações sobre processos e outras estatísticas do sistema através desse diretório. Isso é especialmente útil quando se está trabalhando em um ambiente chroot ou em um sistema isolado, pois fornece uma visão virtual dos processos e do estado do sistema dentro desse ambiente.

```shell

mount -t sysfs none /sys 


mount -t  tmpfs none /tmp 

```

listar uma lista de namespaces

```shell

lslns

```

# Descomplicando o Cgroups

## O que é o Cgroups?

O Cgroups é um recurso do kernel Linux que permite limitar e isolar o uso de recursos de um grupo de processos. Ele pode ser usado para limitar o uso de CPU, memória, E/S de disco e rede de um grupo de processos. O Cgroups também pode ser usado para isolar processos em um ambiente de sistema de arquivos separado, permitindo que você crie ambientes de sistema de arquivos isolados para processos específicos.

36177

```shell

cgcreate -g cpu,memory,blkio,devices,freezer:giropops

```

O comando que você forneceu está criando um grupo de controle chamado "giropops" com os seguintes controladores:

1. `cpu`: Este controlador permite limitar o uso de CPU de um grupo de processos.
2. `memory`: Este controlador permite limitar o uso de memória de um grupo de processos.
3. `blkio`: Este controlador permite limitar o uso de E/S de disco de um grupo de processos.
4. `devices`: Este controlador permite limitar o acesso a dispositivos de um grupo de processos.
5. `freezer`: Este controlador permite congelar e descongelar um grupo de processos.

Portanto, o comando cgcreate -g cpu,memory,blkio,devices,freezer:giropops está criando um novo grupo de controle chamado "giropops" e associando a ele os subsistemas de recursos especificados (CPU, memória, E/S de bloco, dispositivos e freezer), permitindo assim que você aplique restrições e limites específicos a processos que pertencem a esse grupo de controle.

```shell
cgclassify -g cpu,memory,blkio,devices,freezer:giropops 36177
```

O comando que você forneceu está associando o processo com o ID 36177 ao grupo de controle "giropops" com os seguintes controladores:

1. `cpu`: Este controlador permite limitar o uso de CPU de um grupo de processos.
2. `memory`: Este controlador permite limitar o uso de memória de um grupo de processos.
3. `blkio`: Este controlador permite limitar o uso de E/S de disco de um grupo de processos.
4. `devices`: Este controlador permite limitar o acesso a dispositivos de um grupo de processos.
5. `freezer`: Este controlador permite congelar e descongelar um grupo de processos.

```shell
cgset -r cpu.cfs_quota_us=1000 giropops
```

O comando que você forneceu está definindo o limite de uso de CPU do grupo de controle "giropops" para 1000 milissegundos por segundo. Isso significa que os processos que pertencem a esse grupo de controle só poderão usar a CPU por 1000 milissegundos por segundo. Se o processo tentar usar mais do que isso, ele será interrompido e colocado em espera até que o tempo de espera seja atingido.


A flag **-r** é usada para definir um recurso específico para um grupo de controle. Neste caso, o recurso sendo definido é o limite de uso de CPU, que é definido para 1000 milissegundos por segundo.


```shell
dd if=/dev/zero of=catota.img bs=8k count=256k
```

O comando que você forneceu está criando um arquivo de imagem de 2 GB chamado "catota.img" usando o comando dd. O comando dd é usado para copiar arquivos ou dados de um local para outro. Neste caso, o comando está copiando dados do dispositivo de bloco "zero" para o arquivo de imagem "catota.img". O dispositivo de bloco "zero" é um dispositivo de bloco virtual que gera dados aleatórios quando lido. Portanto, o comando dd está copiando dados aleatórios do dispositivo de bloco "zero" para o arquivo de imagem "catota.img".

# Copy-on-Write

## O que é Copy-on-Write?

Copy-on-Write é uma técnica de gerenciamento de memória que permite que vários processos compartilhem a mesma memória física até que um deles modifique os dados. Quando um processo modifica os dados, ele cria uma cópia dos dados em uma nova área de memória e atualiza o ponteiro para apontar para a nova área de memória. Isso permite que vários processos compartilhem a mesma memória física até que um deles modifique os dados.

Antes de entender as camadas propriamente ditas, precisamos entender como um dos principais requisitos para essa coisa acontecer, o Copy-On-Write (ou COW para os íntimos), funciona. Nas palavras do próprio Jérome Petazzoni:

***It's a little bit like having a book. You can make notes in that book if you want, but each time you approach the pen to the page, suddenly someone shows up and takes the page and makes a xerox copy and hand it back to you, that's exactly how copy on write works.***

Em tradução livre, seria como se você tivesse um livro e que fosse permitido fazer anotações nele caso quisesse, porém, cada vez que você estivesse prestes a tocar a página com a caneta, de repente alguém aparecesse, tirasse uma xerox dessa página e entregasse a cópia para você. É exatamente assim que o Copy-On-Write funciona.

Basicamente, significa que um novo recurso, seja ele um bloco no disco ou uma área em memória, só é alocado quando for modificado.

Tá, mas o que isso tudo tem a ver com o Docker? Bom, como você sabe, o Docker usa um esquema de camadas, ou layers, e para montar essas camadas são usadas técnicas de Copy-On-Write. Um container é basicamente uma pilha de camadas compostas por N camadas read-only e uma, a superior, read-write.


# Docker e sua relação com o Kernel Linux

## O que é o Docker?

O Docker é uma plataforma de código aberto para desenvolvimento, envio e execução de aplicativos. Ele permite que você empacote um aplicativo com todas as suas dependências em um contêiner virtualizado que pode ser executado em qualquer sistema operacional Linux. O Docker usa o kernel Linux para fornecer isolamento de recursos e segurança para aplicativos em execução em contêineres.

- O Docker utiliza algumas features básicas do kernel Linux para seu funcionamento. A seguir temos um diagrama no qual é possível visualizar os módulos e features do kernel de que o Docker faz uso:

# Instalando no Debian/Centos/Ubuntu/Suse/Fedora

A instalação do Docker é bastante simples. Você pode optar por instalá-lo utilizando os pacotes disponíveis para sua distro -- por exemplo, o apt-get ou yum.

Nós preferimos fazer a instalação através da execução do curl a seguir, que irá executar um script e detectará qual a distribuição que estamos utilizando, para então adicionar o repositório oficial do Docker em nosso gerenciador de pacotes, o rpm ou apt, por exemplo.

```shell   

curl -fsSL https://get.docker.com/ | sh

```

Assim ele sempre buscará a versão mais recente do Docker. :)

# Criando e gerenciando containers os primeiros containers

```shell

docker container run -it ubuntu

ps -ef

uptime

# Sair do container sem matar ele

CTRL + P + Q


# Voltar para o container

docker conatiner attach <container_id>

# ou

docker conatiner attach <nome_container>


docker container run --name meu_container -it ubuntu

```

O comando `watch` é uma ferramenta no Linux que permite executar outros comandos periodicamente e exibir suas saídas de maneira contínua na linha de comando. O argumento principal do `watch` é o intervalo de tempo (em segundos) em que ele deve executar o comando especificado.

No comando que você forneceu:

```bash
watch -n 1 date
```

Estamos utilizando o `watch` para executar o comando `date` a cada 1 segundo (`-n 1`). Aqui está o que cada parte do comando faz:

- `watch`: O comando que estamos usando.
- `-n 1`: Isso especifica o intervalo de tempo em segundos. Neste caso, `1` segundo, o que significa que o `watch` executará o comando `date` a cada segundo.
- `date`: Este é o comando que retorna a data e hora atuais.

Portanto, quando você executa esse comando, você verá a saída atualizada da hora atual sendo exibida na linha de comando a cada segundo. Isso é útil para monitorar a passagem do tempo ou para ver como um comando se comporta em intervalos regulares. O `watch` continuará a executar até que você o interrompa pressionando `Ctrl + C`.

**Pausando Containers**

```shell

docker container pause <nome_container>

```

**Despausando o container**

```shell

docker container unpause <nome_container>

```

# V
