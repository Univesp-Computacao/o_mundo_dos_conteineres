# Iniciando o Swarm

Bem, chegamos a um ponto importante aqui que é a criação de uma rede de computadores, mas criar uma rede não é tão simples, mesmo que fizesse o passo a passo, vai levar muito tempo com muitas variáveis que podem dar errado, então para conseguimos usar o docker swarm vamos usar o [Play With Docker](https://labs.play-with-docker.com/#){:target="_blank"}

Vai facilitar demais e vamos conseguir subir imagens normalmente, ele já vem com o docker instalado.

1. Aperte **Start**
2. Ele vai solicitar seu login do docker, sua conta criado quando começamos a usar o docker hub.
3. Em seguida se ele não redirecionar basta aperta **Start** novamente.
4. Assim que ele abrir a página, não tenha nenhuma aplicação no momento, basta ficar tentando até conseguir acessar.
5. Quando a aplicação subir vamos ter essa pagina:

![Play](https://i.postimg.cc/R0gVmH3C/image.png){ loading=lazy }

Vamos adicionar 5 instancias, basta pressionar o **+ADD NEW INSTANCE** que ele vai gerar automaticamente os **computadores**:

![Exemplo](https://i.postimg.cc/g02NSnfb/image.png){ loading=lazy }

## Primeiro Gerente

Para iniciar o swarm, usaremos o [docker swarm init](https://docs.docker.com/engine/reference/commandline/swarm_init/){:target="_blank"}.

```bash
docker swarm init
```

Escolha umas das máquinas para roda o comando e observe a saída:

![image do erro](https://i.postimg.cc/P5W53q1y/image.png){ loading=lazy }

Nesse erro, ele nos diz que não anunciamos o IP que devemos usar, o IP que as maquinas estão na rede, para isso vamos usar a variável que ele solicita  **--advertise-addr** e passar o IP, é importante sempre passar essa flag e o IP de umas das máquinas, para fixamos a máquina que criou o swarm.

```bash
docker swarm init --advertise-addr 192.168.0.13
```

E iniciamos o nosso swarm:

![Swarm Feito](https://i.postimg.cc/Prq7VVZR/image.png){ loading=lazy }

A maquina que inicializa o comando, automaticamente se torna líder, ou seja, já temos nosso primeiro gerente.

Nesse momento, ele vai passar dois comandos para que possamos adicionar novos gerentes ou novos trabalhadores:

1. **Gerentes**: [docker swarm join](https://docs.docker.com/engine/reference/commandline/swarm_join-token/){:target="_blank"} com a flag **-token manager**
2. **Trabalhadores**: **docker swarm join** mais a flag **--token SWMTKN-1-317lss7q287qzea30syrpzsnb7dlzcp5xqppqomkz0dpwlf0jv-3qzxl99iwvxo1b7zr2463xgdf 192.168.0.13:2377** passando o token e o IP para os novos computadores se unirem ao swarm.

Antes de continuamos, vamos imaginar que você tenha deixado o ambiente rodando lá e ficou um ano sem acessar, ai um belo dia você volta contudo não lembra qual maquina é a que iniciou o swarm ou se ela está no swarm, para isso vamos usar o [docker info](https://docs.docker.com/engine/reference/commandline/info/){:target="_blank"}.

```bash
docker info
```

E a saída vai ser ± essa aqui:

??? note "Saida"
    ```bash
    Client:
     Context:    default
     Debug Mode: false
     Plugins:
      app: Docker App (Docker Inc., v0.9.1-beta3)

    Server:
     Containers: 0
      Running: 0
      Paused: 0
      Stopped: 0
     Images: 0
     Server Version: 20.10.0
     Storage Driver: overlay2
      Backing Filesystem: xfs
      Supports d_type: true
      Native Overlay Diff: true
     Logging Driver: json-file
     Cgroup Driver: cgroupfs
     Cgroup Version: 1
     Plugins:
      Volume: local
      Network: bridge host ipvlan macvlan null overlay
      Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
     Swarm: active
      NodeID: tv5szgmtbqunlbhkv4jqqdjrz
      Is Manager: true
      ClusterID: qxz7jhjgyd5ldoy7o1vcutbfn
      Managers: 1
      Nodes: 1
      Default Address Pool: 10.0.0.0/8  
      SubnetSize: 24
      Data Path Port: 4789
      Orchestration:
       Task History Retention Limit: 5
      Raft:
       Snapshot Interval: 10000
       Number of Old Snapshots to Retain: 0
       Heartbeat Tick: 1
       Election Tick: 10
      Dispatcher:
       Heartbeat Period: 5 seconds
      CA Configuration:
       Expiry Duration: 3 months
       Force Rotate: 0
      Autolock Managers: false
      Root Rotation In Progress: false
      Node Address: 192.168.0.13
      Manager Addresses:
       192.168.0.13:2377
     Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
     Default Runtime: runc
     Init Binary: docker-init
     containerd version: 269548fa27e0089a8b8278fc4fc781d7f65a939b
     runc version: ff819c7e9184c13b7c2607fe6c30ae19403a7aff
     init version: de40ad0
     Security Options:
      apparmor
      seccomp
       Profile: default
     Kernel Version: 4.4.0-210-generic
     Operating System: Alpine Linux v3.12 (containerized)
     OSType: linux
     Architecture: x86_64
     CPUs: 8
     Total Memory: 31.42GiB
     Name: node1
     ID: MLNO:DP7B:WIP5:4LLV:IUGJ:B5DP:KMRL:PZ2Z:A7Y3:7XLT:Z4NJ:UULR
     Docker Root Dir: /var/lib/docker
     Debug Mode: true
      File Descriptors: 38
      Goroutines: 155
      System Time: 2022-03-05T23:02:27.17700525Z
      EventsListeners: 0
     Registry: https://index.docker.io/v1/
     Labels:
     Experimental: true
     Insecure Registries:
      127.0.0.1
      127.0.0.0/8
     Live Restore Enabled: false
     Product License: Community Engine
    ```

E nessa lista todas temos o **Swarm: active**, significa que o swarm está ativo e o **Is Manager: true** que diz que a maquina é um gerente.

## Primeiro Trabalhador

Agora que temos nosso swarm, vamos inserir mais computadores, vamos usar o [docker swarm join](https://docs.docker.com/engine/reference/commandline/swarm_join/){:target="_blank"}

Nas outras instancias que você tiver, basta copia o comando com shift+insert que o docker fornece, no meu caso é:

```bash
docker swarm join --token SWMTKN-1-317lss7q287qzea30syrpzsnb7dlzcp5xqppqomkz0dpwlf0jv-3qzxl99iwvxo1b7zr2463xgdf 192.168.0.13:2377
```

E a saida:

```bash
This node joined a swarm as a worker.
```

Agora faça isso em todas as instâncias que você criou, recomendo não passar de 5 por questão de estabilidade e desempenho do serviço.

Se você perde o token, não precisa anotar nenhum comando, basta no ir ao terminal do gerente e pergunta para ele, usando o comando [docker swarm join-token](https://docs.docker.com/engine/reference/commandline/swarm_join-token/){:target="_blank"} e passe quem você quer saber, worker ou maneger

```bash
docker swarm join-token worker
```

## Nó - Nodes

Precisamos ver de uma forma geral nosso swarm, para isso vamos usar o comando [docker node ls](https://docs.docker.com/engine/reference/commandline/node_ls/){{:target="_blank"}, no gerente, se usar no trabalhador, vai dá um erro.

```bash
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
tv5szgmtbqunlbhkv4jqqdjrz *   node1      Ready     Active         Leader           20.10.0
somcfzr1i52h02s32g8162jv5     node2      Ready     Active                          20.10.0
m3ug6hoe28zz1dap3ejhqug6n     node3      Ready     Active                          20.10.0
le62bq94ze1eagudy8dtx12ed     node4      Ready     Active                          20.10.0
aedfo2gsp1ky5cbzb7vmj1eqy     node5      Ready     Active                          20.10.0
```

Explicando:

+ **ID**: É o ID do nó, e o nó com * é o nó que iniciou o swarm.
+ **HOSTNAME**: Nome das máquinas
+ **STATUS**: Que está funcionando normalmente
+ **AVAILABILITY**: Que vai rodar contêineres normalmente
+ **MANAGER STATUS**: Quem são os gerentes e o líder, todos os, nó sem informação, vazio, são workes para o docker.
+ **ENGINE VERSION**: Engine é a versão do docker.

Para exemplificar, vamos roda o comando em um trabalhador:

```bash
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
```

Então todos os comandos de alteração ou visualização, somente em nó gerente. Por exemplo, vamos remover um nó do cluster, usando o [docker node rm](https://docs.docker.com/engine/reference/commandline/node_rm/){:target="_blank"}, basta passar o ID do nó que você quer derrubar.

=== "Execute"

    ```bash
    docker node rm aedfo2gsp1ky5cbzb7vmj1eqy
    ```

=== "Saida"

    ```bash
    Error response from daemon: rpc error: code = FailedPrecondition desc = node aedfo2gsp1ky5cbzb7vmj1eqy is not down and can't be removed
    ```

O erro está dizendo que o nó que tentamos remover não está com o status **down**, para resolver isso precisamos ir ao nó que queremos derrubar e usar o [docker swarm leave](https://docs.docker.com/engine/reference/commandline/swarm_leave/){:target="_blank"}para muda o status do nó.


=== "Execute"

    ```bash
    docker swarm leave
    ```

=== "Saida"

    ```bash
    Node left the swarm.
    ```

Retorne para o nó gerente:

    ```bash
    docker node ls
    ```

=== "Saida"

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    tv5szgmtbqunlbhkv4jqqdjrz *   node1      Ready     Active         Leader           20.10.0
    somcfzr1i52h02s32g8162jv5     node2      Ready     Active                          20.10.0
    m3ug6hoe28zz1dap3ejhqug6n     node3      Ready     Active                          20.10.0
    le62bq94ze1eagudy8dtx12ed     node4      Ready     Active                          20.10.0
    aedfo2gsp1ky5cbzb7vmj1eqy     node5      Down      Active                          20.10.0
    ```

O **status** do nó 5 mudou para **down** e com isso podemos remover efetivamente o nó.

```bash
docker node rm aedfo2gsp1ky5cbzb7vmj1eqy
```

E de outro **docker node ls** para confirmar.

Adicione o nó novamente para continuamos, use o [docker swarm join](https://docs.docker.com/engine/reference/commandline/swarm_join/){:target="_blank"}
