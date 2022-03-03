## Rede Bridge - Ponte

A rede, uma rede de ponte é um dispositivo de camada de link que encaminha o tráfego entre os segmentos de rede. Uma ponte pode ser um dispositivo de hardware ou um dispositivo de software executado no kernel de uma máquina host.

No Docker, uma rede de ponte usa uma ponte de software que permite que os contêineres conectados à mesma rede de ponte se comuniquem, enquanto fornece isolamento de contêineres que não estão conectados a essa rede de ponte. O driver de ponte do Docker instala automaticamente as regras na máquina host para que os contêineres em diferentes redes de ponte não possam se comunicar diretamente entre si.

As redes de ponte se aplicam a contêineres em execução no mesmo host do daemon do Docker. Para comunicação entre contêineres em execução em diferentes hosts do daemon do Docker, você pode gerenciar o roteamento no nível do sistema operacional ou usar uma rede de sobreposição.

Quando você inicia o Docker, uma rede de ponte padrão (também chamada Bridge) é criada automaticamente e os contêineres recém-iniciados se conectam a ela, a menos que especificado de outra forma. Você também pode criar redes de ponte personalizadas definidas pelo usuário. As redes de ponte definidas pelo usuário são superiores à rede de ponte padrão.

![Docker Isolamento](https://i.postimg.cc/BQgqDjZn/esquema-docker-bridge.png){ loading=lazy }

Realizaremos um teste:

1. execute o **docker run -it ubuntu bash**, deixei o terminal aberto e abra uma nova aba no terminal
2. Nesse terminal execute **docker ps**, copie o id do contêiner e vamos usar o [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/){:target="_blank"}

```bash
docker inspect 8c9e5c315c0f
```

A saída vai ser um JSON array e lá no final vai esta as informações sobre a rede:

```JSON
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "f2265fd0f92380df4ff74210e72ae304c76b3720cfb07bb134d97aa6ed215d70",
                    "EndpointID": "86e57b1d3e2f1b3019246d6b50da7d88744c6bdc4acdb62f8b69236d1407185b",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
```

Entre todas essas configurações, temos o **NetworkID** que é a rede do **docker0**, ou seja, por padrão o docker mantém os mesmo para todos os containers **NetworkID**.

Execute o [docker network ls](https://docs.docker.com/engine/reference/commandline/network_ls/){:target="_blank"} para verificar a saida:

```bash
NETWORK ID     NAME      DRIVER    SCOPE
f2265fd0f923   bridge    bridge    local
95dc365e7cbf   host      host      local
093d70e2475c   none      null      local
```

+ **NETWORK ID**: Id da rede
+ **NAME**: Nome da rede
+ **DRIVER**: Driver da rede
+ **SCOPE**: É o alcance, com as opções de local ou global

Para mais informações [docker network](https://docs.docker.com/engine/reference/commandline/network/){:target="_blank"}.

----

## Testando a rede Bridge

Para isso, vamos usar dois contêineres ubuntu:

=== "Ubuntu 1"

    ```bash
    docker run -it ubuntu bash
    ```

    + Quando o bash iniciar:

    ```bash
    apt-get update
    ```

    + E vamos instalar o ping

    ```bash
    apt-get install iputils-ping -y
    ```

=== "Ubuntu 2"

    ```bash
    docker run -it ubuntu bash
    ```

    + Assim que o bash iniciar deixei esse ubuntu de lado

----

1. Precisamos do IP dos contêineres, para isso vamos usar **docker ps** e em seguida o [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/){:target="_blank"}
2. Geralmente o **Gateway** é **172.17.0.1** e os contêineres continuar a numeração. Faça uma anotação de qual é o contêiner/IP para usar.
3. Use no ubuntu 1 o comando **ping** e passe o IP do ubuntu 2.
4. No meu caso a saída ficou assim:

```bash
root@8c9e5c315c0f:/# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.313 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.147 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.123 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.151 ms
```

Para finalizar a execução basta aperta ctrl + c, e acabamos de usar a rede do docker para realizar a comunicação entre dois contêineres.

## Melhorando a comunicação

Bem, deve ter percebido que ficar procurando IP dos contêineres e anotando cada IP não é uma boa ideia, além de que os contêineres podem perder a conexão ou cair, pensando nesse problema o docker criou uma coluna **NAME** no contêineres e com ela que vamos resolver todo esse problema. Vamos passa a flag **--name** durante o docker run para nomear nossos contêineres e vamos criar nossa rede, é um processo bem simples:

Usaremos o [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/){:target="_blank"} para criar uma rede, vamos usar a flag **--driver** para passar o driver bridge.

```bash
docker network create --driver bridge rede-teste
```

E vamos passar nossa rede na criação do contêiner:

=== "Pong"

    ```bash
    docker run -it --name pong --network rede-teste ubuntu sleep 30m
    ```


=== "Ping"

    ```bash
    docker run -it --name ubuntu1 --network rede-teste ubuntu bash
    ```

    + Quando o bash iniciar:

    ```bash
    apt-get update
    ```

    + E vamos instalar o ping

    ```bash
    apt-get install iputils-ping -y
    ```

    + Execute o ping, mas agora vamos passar o nome do contêiner e não o IP

    ```bash
    ping pong
    ```


----

A diferença entre o padrão do docker [bridge network](https://docs.docker.com/network/bridge/){:target="_blank"} e o user-defined bridges é que quando criamos nossa própria rede bridge, no processo é provido uma solução DNS automática nos contêineres, possibilitando esse teste que fizemos, sem ocorrer a necessidade de mais configurações.

Quando executamos o [docker network ls](https://docs.docker.com/engine/reference/commandline/network_ls/){:target="_blank"}, verificamos que temos mais duas categorias de drivers de redes disponíveis, **none** e **host**.

A rede com o driver **none** basicamente não criar conexões no contêiner, removendo a interface de rede:

```bash
docker run -d --network none ubuntu sleep 30m
```

De um **docker inspect** nesse contêiner e o resultado:

```JSON
            "Networks": {
                "none": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "093d70e2475c6ddd793426f4f6d3aa5560fd946ae182219aff56b2f406f1d290",
                    "EndpointID": "59be8bf2afca5f2a103aa48fe86a85a676dd8ffe48a4d1101d128dcb5e9ad8b1",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "",
                    "DriverOpts": null
                }
            }
```

Os campos estão sem informação de IP ou estão com null. Isolando completamente o contêiner.

Enquanto o driver **host**, elimina o isolamento proporcionado pela estrutura do docker, ou seja, acessa a rede diretamente da host machine:

```bash
docker run -d --network host sposigor/app-flask-teste:1
```

Se lembra que no projeto o flask define a porta 5000, e para acessar no navegador era preciso passa a flag **-p** para redirecionar? Com o driver **host** da rede, basta acessar diretamente, copie e cole no navegador:

```bash
localhost:5000
```

Para lembrar ainda é possível ter problemas de portas, ou seja, se houve mais alguma aplicação usando a porta 5000 vai ocorrer um erro.
