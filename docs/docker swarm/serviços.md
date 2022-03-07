# Serviço

Antes de subimos nossa primeira imagem, vamos realizar um teste, para entender melhor o fluxo que devemos seguir quando usamos o swarm:

1. Escolha um dos nós trabalhadores
2. Execute o comando **docker run -d -p 5000:5000 sposigor/app-flask-teste:1**
3. Libere a porta em **OPEN PORT** e digite **5000**:

![porta](https://i.postimg.cc/FFSX4Sth/image.png){ loading=lazy }

Ele vai abrir uma nova aba com a aplicação.

Se lembra de como funciona o swarm? Então, escolha outro nó trabalhador e execute o **docker ps**, faça o mesmo para o nó gerente e em ambos vamos ter esse resultado:

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Estranho né, se o swarm cria essa conexão com todos, porque o contêiner que subimos não foi refletido nos demais?

> É porque rodamos uma imagem local, e não no modo swarm.

Certo, o modo swarm precisa de um comando mais específico para roda a imagem que é o [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/){:target="_blank"}.

Antes de executar, vamos remover a imagem local, vá ao terminal do nó que a imagem está rodando:

```bash
docker rm 56e3abd7f344 --force
```

E agora sim, vamos criar nosso serviço:

=== "Execute"

    ```bash
    docker service create -p 5000:5000 sposigor/app-flask-teste:1
    ```

    + A diferença é somente o comando inicial, **service create**, que vai permitir que todos os nós se comuniquem no modo swarm.

=== "Saida"

    ```bash
    Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
    ```

----

Se você executou o comando em um nó trabalhador, vai receber o erro do qual os trabalhadores não podem iniciar um serviço, apesar do docker run executar, para subir uma imagem no modo swarm é preciso fazer através do gerente.

Vá ao nó gerente e execute o comando.

=== "Execute"

    ```bash
    docker service create -p 5000:5000 sposigor/app-flask-teste:1
    ```

=== "Saida"

    ```bash
    9q3asggjagwfw89vrojc1urew
    overall progress: 1 out of 1 tasks
    1/1: running   [==================================================>]
    verify: Service converged
    ```

----

Então ele está informando que o **Service converged**, mas o que isso significa? Basicamente que ocorreu bem o processo de criação de um serviço e com isso ele criou uma tarefa: **overall progress: 1 out of 1 tasks**.

Vamos por partes, execute o [docker servie ls](https://docs.docker.com/engine/reference/commandline/service_ls/){:target="_blank"}.

=== "Execute"

    ```bash
    docker service ls
    ```

    + Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID             NAME           MODE         REPLICAS   IMAGE                        PORTS
    9q3asggjagwf   epic_almeida   replicated   1/1        sposigor/app-flask-teste:1   *:5000->5000/tcp
    ```

----

Analisar o cabeçalho para entender o que significa cada coluna:

+ **ID**: É o ID do serviço
+ **NAME**: É o nome do serviço
+ **MODE**: É o modo do serviço
+ **REPLICAS**: É a quantidade de replicas
+ **IMAGE**: É a imagem que o serviço está rodando
+ **PORTS**: É a porta que está disponivel

Para saber qual maquina está executando o container, use o [docker service ps](https://docs.docker.com/engine/reference/commandline/service_ps/){:target="_blank"}, e passe o ID do serviço.

=== "Execute"

    ```bash
    docker service ps 9q3asggjagwf
    ```

    + Lembrando só o nó gerente pode executar
    + Se quiser como é possivel passar só alguns caracteres **docker service ps 9q3**, desde que não tenha outro serviço com o mesmo inicio.

=== "Saida"

    ```bash
    ID             NAME             IMAGE                        NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
    3qpfhbbmigm4   epic_almeida.1   sposigor/app-flask-teste:1   node2     Running         Running 28 minutes ago      
    ```

----

Analisar o cabeçalho para entender o que significa cada coluna:

+ **ID**: É o ID do serviço.
+ **NAME**: É o nome do serviço.
+ **IMAGE**: É a imagem que o serviço está rodando.
+ **NODE**: É o nó que está executando a imagem naquele momento.
+ **DESIRED STATE**: É o estado desejado da tarefa, pode está em (running, shutdown, or accepted).
+ **CURRENT STATE**: É o estado atual da tarefa.
+ **ERROR**: Se houver algum erro pe informado
+ **PORTS**: É a porta que está disponível.

Bem, agora que o serviço está disponível como swarm, vamos testa a aplicação em outras máquinas, vá em um nó trabalhador e libere a porta 5000.

![liberando as portas](https://media4.giphy.com/media/LE9a8WL8OCODLKtiw9/giphy.gif?cid=790b7611c5ce97dee2466ecf552a7a2dd243473e40b47250&rid=giphy.gif&ct=g){ loading=lazy }

Por mais que apenas uma das máquinas esteja rodando a aplicação, ela está disponível na porta 5000 de todas as maquinas. Quem é que o cara que está ajudando a resolver esse problema é o [Routing Mesh](https://docs.docker.com/engine/swarm/ingress/){:target="_blank"} através do Load balancing.

----

## Routing Mesh

O modo swarm do Docker Engine facilita a publicação de portas para serviços para disponibilizá-los para recursos fora do swarm. Todos os nós, ficam disponíveis em uma malha de roteamento de entrada. A malha de roteamento permite que cada nó no enxame aceite conexões em portas publicadas para qualquer serviço em execução no enxame, mesmo que não haja nenhuma tarefa em execução no nó. A malha de roteamento roteia todas as solicitações de entrada para portas publicadas em nós disponíveis para um contêiner ativo.

![Routing Mesh](https://i.postimg.cc/HnfG7WXf/image.png){ loading=lazy }

Testar na prática, vamos subir a mesma imagem na mesma porta para ver o que ocorre:

=== "Execute"

    ```bash
    docker service create -p 5000:5000 sposigor/app-flask-teste:1
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    Error response from daemon: rpc error: code = InvalidArgument desc = port '5000' is already in use by service 'epic_almeida' (9q3asggjagwfw89vrojc1urew) as an ingress port
    ```

----

E assim ele nos dá um erro, para verificar melhor vamos remover o contêiner que no meu caso está rodando no node2:

=== "Execute"

    ```bash
    docker rm 6f21 --force
    ```

----

Voltei para o nó gerente e execute o **docker service ls**:

=== "Execute"

    ```bash
    docker service ls
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID             NAME           MODE         REPLICAS   IMAGE                        PORTS
    9q3asggjagwf   epic_almeida   replicated   1/1        sposigor/app-flask-teste:1   *:5000->5000/tcp
    ```

----

Está dizendo que o serviço ainda está rodando, vamos usar o [docker service ps](https://docs.docker.com/engine/reference/commandline/service_ps/){:target="_blank"} e passa o ID do serviço:

=== "Execute"

    ```bash
    docker service ps 9q3asggjagwf
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID             NAME                 IMAGE                        NODE      DESIRED STATE   CURRENT STATE           ERROR                         PORTS
    f91ba1oajs39   epic_almeida.1       sposigor/app-flask-teste:1   node3     Running         Running 2 minutes ago                                 
    3qpfhbbmigm4    \_ epic_almeida.1   sposigor/app-flask-teste:1   node2     Shutdown        Failed 3 minutes ago    "task: non-zero exit (137)"
    ```

E o swarm se encarregou de subir a imagem automaticamente em outro nó, no caso do node2 para o node3, então se fomos lá na porta 5000 novamente, a aplicação vai estar rodando normalmente.

----

## Gerente

Imagine que sem querer você desligue o computador que está o gerente? E agora será que deu ruim?

Sim, se você possui somente um gerente no seu swarm, o swarm vai ser desconfigurado, sendo necessário, reconfigurar o swarm. Mas os serviços param? Não, eles vão continuar, contudo, não existe mais ninguém para garantir a estabilidade do serviço.

Como podemos resolver essa categoria de situação? Basta fazer o backup da pasta do swarm.

Isso vai ser feito de forma manual, porque o docker não disponibiliza ainda nenhuma ferramenta para backup nesse sentido.

1. Faça uma cópia da sua pasta que contém as informações do swarm, no Linux ela fica em **/var/lib/docker/swarm/**
2. No Linux basta usar o comando **cp -r /var/lib/docker/swarm/ backup**

Agora com seu backup feito, para subir um swarm do zero com as configurações nesse backup:

1. Vamos fazer o caminho inverso, **cp -r backup/* /var/lib/docker/swarm/**
2. Durante a criação do swarm vamos passa uma nova flag **--force-new-cluster**
3. **docker swarm init --force-new-cluster --advertise-addr 192.168.0.13**

Essa é a fórmula mais usada para recuperação de swarm, independente do que tenha acontecido. Numa aplicação pequena, não faz sentido ter que criar esses backups, porém quando os serviços são muitos e a infraestrutura é maior ainda, é necessário ter garantias para recuperar rapidamente o estado anterior ao problema.

----

### Criando Gerentes

Talvez, ter mais gerentes no seu swarm possa ajudar a resolve esse problema, afinal por mais que o backup seja uma solução, para determinadas situações, podemos simplesmente adicionar novos Gerentes.

Adicione mais uma instância no docker play e vamos passar para o token do comando [docker swarm join-token](https://docs.docker.com/engine/reference/commandline/swarm_join-token/){:target="_blank"} e passe o manager:

=== "Execute"

    ```bash
    docker swarm join-token manager
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    To add a manager to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-1waduqjabmpmygpy6xicwtt1srwbchelwnu8fsv3fj8qk4jdvi-8tke9gy4e6crw0euq2tki1m0y 192.168.0.13:2377
    ```

----

Vá à instância criada e execute o comando passado.

=== "Execute"

    ```bash
    docker swarm join --token SWMTKN-1-1waduqjabmpmygpy6xicwtt1srwbchelwnu8fsv3fj8qk4jdvi-8tke9gy4e6crw0euq2tki1m0y 192.168.0.13:2377
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    This node joined a swarm as a manager.
    ```

----

Se por acaso estive com a configuração de 1 gerente e 4 trabalhadores, basta derruba um dos trabalhadores e remover do swarm, e em seguida suba novamente como gerente.

Em um dos gerentes:

=== "Execute"

    ```bash
    docker node ls
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    fhiexwgtupwto0yzz5i4nbve7     node1      Ready     Active         Leader           20.10.0
    x3ghs9kh5nftbnnd7otoyg4hu     node2      Ready     Active                          20.10.0
    nufnx3dlv0orjgb0dm5li4aah     node3      Ready     Active                          20.10.0
    jvxn6dt0zob66po3d5rdrbaru     node4      Ready     Active                          20.10.0
    1bk98ofc3qasq9n6v298x4jjv *   node5      Ready     Active         Reachable        20.10.0
    ```

E pronto o seu novo gerente já está rodando normalmente.

----

### Raft na prática

Vamos reiniciar nossa aplicação no docker play feche com o close session e de outro start, lembrando dependendo o serviço pode estar cheio, mas basta continuar tentando até conseguir.

Click na chave de boca e selecione a opção:

![Selecione](https://i.postimg.cc/Y9WdtRZk/image.png){ loading=lazy }

Após carregar a configuração, execute **docker node ls** em qualquer um dos gerentes:

=== "Execute"

    ```bash
    docker node ls
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    3g4vyx694h4pd3dxlj442x88z     manager1   Ready     Active         Reachable        20.10.0
    lheo3at6gjkka6uoehk3lr0ow *   manager2   Ready     Active         Leader           20.10.0
    ptvwj6n2ums5o8b0lsfabhxgk     manager3   Ready     Active         Reachable        20.10.0
    n2230i73ivqhyqqg0ty2hw3nd     worker1    Ready     Active                          20.10.0
    tdiufxyidqv47atj937ffthb1     worker2    Ready     Active                          20.10.0
    ```

----

Agora temos 3 gerentes, sendo um deles o lider. Vamos derrubar o lider com **docker swarm leave --force** no terminal do lider.

Assim que executar vá em outro gerente:

=== "Execute"

    ```bash
    docker node ls
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    3g4vyx694h4pd3dxlj442x88z *   manager1   Ready     Active         Leader           20.10.0
    lheo3at6gjkka6uoehk3lr0ow     manager2   Unknown   Active         Unreachable      20.10.0
    ptvwj6n2ums5o8b0lsfabhxgk     manager3   Ready     Active         Reachable        20.10.0
    n2230i73ivqhyqqg0ty2hw3nd     worker1    Ready     Active                          20.10.0
    tdiufxyidqv47atj937ffthb1     worker2    Ready     Active                          20.10.0
    ```

----

Temos um novo líder agora, que pe maneger1, enquanto o manager3 está com o **MANAGER STATUS** de **Unreachable**, ou seja, está fora do swarm por algum motivo.

O que ocorreu? O Raft, foi acionado para gera uma eleição, e o vencedor da eleição foi o maneger1.

Recomendação do docker sobre os gerentes, eles recomendam que tenhamos números impares, 3, 5 ou 7.

----

### Removendo Gerentes

Vamos remover o nosso gerente que está **Unreachable**. Em qualquer gerente execute:

=== "Execute"

    ```bash
    docker node rm manager2
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    Error response from daemon: rpc error: code = FailedPrecondition desc = node lheo3at6gjkka6uoehk3lr0ow is a cluster manager and is a member of the raft cluster. It must be demoted to worker before removal
    ```

----

E ele deu um erro, ele está dizendo que precisamos rebaixar o menager2 para trabalhador antes de remover efetivamente do swarm, usando o [docker node demote](https://docs.docker.com/engine/reference/commandline/node_demote/){ loading=lazy }:

=== "Execute"

    ```bash
    docker node demote manager2
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    Manager manager2 demoted in the swarm.
    ```

Agora é possível remover com o **docker node rm**, faça e vamos adicionar o gerente no swarm com o **docker swarm join-token manager** pegue o comando que ele retorna e cola no gerente que você removeu.

----
## Restrição de Serviços

Por padrão o docker permite que o gerente rode serviços, ou seja, é possível que a maquina que vai efetivamente roda alguma imagem seja um gerente e não um trabalhador. Mas vamos supor que por algum motivo você não quer permitir que o gerente rode qualquer serviço, ou só quer determinado serviço rode no gerente.

Então podemos resolver da seguinte forma:

=== "1"

    Suba a imagem

    ```bash
    docker service create -p 5000:5000 sposigor/app-flask-teste:1
    ```

    Lembrando só o nó gerente pode executar

=== "2"

    ```bash
    docker node ls
    ```

    E a saída:

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    3g4vyx694h4pd3dxlj442x88z     manager1   Ready     Active         Leader           20.10.0
    3xtgnz4d2098nnhjrq5ilv7l6 *   manager2   Ready     Active         Reachable        20.10.0
    ptvwj6n2ums5o8b0lsfabhxgk     manager3   Ready     Active         Reachable        20.10.0
    n2230i73ivqhyqqg0ty2hw3nd     worker1    Ready     Active                          20.10.0
    tdiufxyidqv47atj937ffthb1     worker2    Ready     Active                          20.10.0
    ```

=== "3"

    Aqui vamos alterar o estado da **AVAILABILITY**, para conseguimos fazer essa restrição do nó.
    Enquanto estiver como **Active** significa que está disponível para roda serviços.
    Para alterar vamos usar o [docker node update](https://docs.docker.com/engine/reference/commandline/node_update/){:target="_blank"}
    Vamos passa a flag **--availability** que tem 3 estados ("active"|"pause"|"drain").
    No nosso caso queremos usar o **drain** para que o gerente não execute nenhum serviço.
    Por fim basta passa qual maquina você quer alterar o estado.

    ```bash
    docker node update --availability drain manager2
    ```

=== "4"

    Rode novamente o **docker node ls**

    ```bash
    docker node ls
    ```

    Saida:

    ```bash
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    3g4vyx694h4pd3dxlj442x88z     manager1   Ready     Active         Leader           20.10.0
    3xtgnz4d2098nnhjrq5ilv7l6 *   manager2   Ready     Drain          Reachable        20.10.0
    ptvwj6n2ums5o8b0lsfabhxgk     manager3   Ready     Active         Reachable        20.10.0
    n2230i73ivqhyqqg0ty2hw3nd     worker1    Ready     Active                          20.10.0
    tdiufxyidqv47atj937ffthb1     worker2    Ready     Active                          20.10.0
    ```

----

Assim conseguimos alterar a **AVAILABILITY** do **manager2** para a **Drain** e com isso ele se torna inacessivel para executar serviços, mas continuar a executar a atividade de gerente normalmente.

Mas vamos supor que você queira fazer isso em todos os gerentes de uma vez, vamos ter que alterar 1 por 1? Não, vamos usar um comando mais generalista que vai definir aonde queremos que o serviço rode. Usando o [docker service update](https://docs.docker.com/engine/reference/commandline/service_update/){:target="_blank"} e passando a flag **--constraint-add** que vai restrigir determido serviço para um grupo especifico.

=== "Execute"

    Vamos passa para o comando a flag **--constraint-add**
    Nesse caso queremos especificar um tipo de nó, então vamos usar o **node.role==worker**
    E por ultimo vamos passa o ID do serviço para saber qual serviço vamos aplicar essa restrição

    ```bash
    docker service update --constraint-add node.role==worker gnj34gs43ebi
    ```

=== "Saida"

    ```bash
    gnj34gs43ebi
    overall progress: 1 out of 1 tasks
    1/1: running   [==================================================>]
    verify: Service converged
    ```

----

Alguns cuidados durante a restrição desses serviços, vamos supor que você faça essa restrição usando um nó que não existe, ele vai rodar o comando e todas as suas aplicações se tornaram indisponiveis, porque a restrição que houve, restringiu para um nó inexistente.

+ Quando restringir o serviços nos gerente?

Queremos rodar serviços no manager, que são serviços de monitoramento, serviços de segurança, ou seja, serviços que são críticos, que dependemos na nossa aplicação. É importante pondera essa questão, para entender quais são os pontos criticos da aplicação.

----

## Replicas

Já vimos essa palavra aqui:

=== "Execute"

    ```bash
    docker service ls
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID             NAME              MODE         REPLICAS   IMAGE                        PORTS
    gnj34gs43ebi   dazzling_lamarr   replicated   1/1        sposigor/app-flask-teste:1   *:5000->5000/tcp
    ```

----

Certo o que significa isso, bem para explicar a gente precisa descobrir aonde nosso serviço está rodando:

=== "Execute"

    ```bash
    docker service ps gnj34gs43ebi
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    ID             NAME                IMAGE                        NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
    w105x64ylhne   dazzling_lamarr.1   sposigor/app-flask-teste:1   worker2   Running         Running 37 minutes ago      
    ```

----

O serviço está rodando no **worker2**, para explicar vamos repassar pelo conceito de Load Balancing. Quando a porta 5000 é acessada independete da maquina que estiver no swarm, a aplicação é diponibilizada, porém se houver interação, vamos supor que haja um cadastro de um usuario, o que ocorre na pratica é que essa alteração vai ser redirecionada para a maquina que está rodando essa imagem. E a Replica serve para isso, para que possamos usar mais de uma maquina nesse processo de processamento dessa demanda. Então vamos replicar essa imagem, que para outras maquinas com o [docker service update](https://docs.docker.com/engine/reference/commandline/service_update/){:target="_blank"}.

=== "Execute"

    ```bash
    docker service update --replicas 8 gnj34gs43ebi
    ```

    Lembrando só o nó gerente pode executar

=== "Saida"

    ```bash
    gnj34gs43ebi
    overall progress: 4 out of 4 tasks
    1/4: running   [==================================================>]
    2/4: running   [==================================================>]
    3/4: running   [==================================================>]
    4/4: running   [==================================================>]
    verify: Service converged     
    ```

----

Vamos executar o **docker service ls** para verificar como está as réplicas:

=== "docker service ls"

    ```bash
    docker service ls
    ```

    Lembrando só o nó gerente pode executar

    Saida:

    ```bash
    ID             NAME              MODE         REPLICAS   IMAGE                        PORTS
    gnj34gs43ebi   dazzling_lamarr   replicated   4/4        sposigor/app-flask-teste:1   *:5000->5000/tcp
    ```

    Temos agora, 4 replicas

=== "docker service ps"

    Para saber quais maquinas estão rodando o serviço:

    ```bash
    docker service ps gnj34gs43ebi
    ```

    Saida:

    ```bash
    ID             NAME                IMAGE                        NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
    w105x64ylhne   dazzling_lamarr.1   sposigor/app-flask-teste:1   worker2   Running         Running 52 minutes ago             
    v9bmopdi3nr3   dazzling_lamarr.2   sposigor/app-flask-teste:1   worker1   Running         Running 3 minutes ago              
    sp81t2z56n67   dazzling_lamarr.3   sposigor/app-flask-teste:1   worker1   Running         Running 4 minutes ago              
    oxdvxulccegb   dazzling_lamarr.4   sposigor/app-flask-teste:1   worker2   Running         Running 5 minutes ago   
    ```

    Lembrando que fizemo a restrição para somente workes executarem as tarefas.
    Vamos remover a restrição e tentar novamente com 8 replicas

=== "Removendo --constraint-rm"

    + Vamos passar os mesmos parâmetros que usamos para restringir os serviços

    ```bash
    docker service update --constraint-rm node.role==worker gnj34gs43ebi
    ```

=== "8 Replicas"

    + Já vamos passar 8 no parâmetro.

    ```bash
    docker service update --replicas 8 gnj34gs43ebi
    ```

    Em seguida:

    ```bash
    docker service ps gnj34gs43ebi
    ```

    E a saída:

    ```bash
    ID             NAME                IMAGE                        NODE       DESIRED STATE   CURRENT STATE               ERROR     PORTS
    w105x64ylhne   dazzling_lamarr.1   sposigor/app-flask-teste:1   worker2    Running         Running about an hour ago             
    v9bmopdi3nr3   dazzling_lamarr.2   sposigor/app-flask-teste:1   worker1    Running         Running 17 minutes ago                
    sp81t2z56n67   dazzling_lamarr.3   sposigor/app-flask-teste:1   worker1    Running         Running 17 minutes ago                
    oxdvxulccegb   dazzling_lamarr.4   sposigor/app-flask-teste:1   worker2    Running         Running 18 minutes ago                
    gs8m3brh4sw6   dazzling_lamarr.5   sposigor/app-flask-teste:1   manager3   Running         Running 3 minutes ago                 
    sr1u0kr19hut   dazzling_lamarr.6   sposigor/app-flask-teste:1   manager3   Running         Running 3 minutes ago                 
    z80bqt30sscq   dazzling_lamarr.7   sposigor/app-flask-teste:1   manager3   Running         Running 3 minutes ago                 
    2yjmab0lkl7q   dazzling_lamarr.8   sposigor/app-flask-teste:1   manager1   Running         Running 3 minutes ago    
    ```

----

Com isso agora temos diversas maquinas rodando nosso serviço, ou seja, o poder de processamento aumentou. Porque assim que ocorrer uma demanda de processamento, o load balancing por trás vai estar resolvendo isso.

Também é possível usar o [docker service scale](https://docs.docker.com/engine/reference/commandline/service_scale/){:target="_blank"}, vamos passar o ID do serviço e a quantidade de réplicas para escalar

```bash
docker service scale gnj34gs43ebi=6
```

----

Temos também a opção de passar diretamente na criação do serviço, basta passa a flag **--mode** e o parâmetro **global**

```bash
docker service create -p 5000:5000 --mode global sposigor/app-flask-teste:1
```

É importante lembrar que não é recomendado passar todos os serviços com o parâmetro global, afinal isso vai exigir um consumo que poderia ser poupado da aplicação no processador individual. Seja sempre critico com essa necessidade.
