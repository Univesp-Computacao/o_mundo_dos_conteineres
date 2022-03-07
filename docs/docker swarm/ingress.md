# Ingress

Que tal damos uma olhada na rede?

=== "Execute"

    ```bash
    docker network ls
    ```

    Saida:

    ```bash
    NETWORK ID     NAME              DRIVER    SCOPE
    b61db738d8e1   bridge            bridge    local
    a1dc5572a15b   docker_gwbridge   bridge    local
    0310735b3f9c   host              host      local
    k6lk5evd8ifb   ingress           overlay   swarm
    8e6d45cb021f   none              null      local
    ```

=== "Execute em outro nó"

    ```bash
    docker network ls
    ```

    Saida:

    ```bash
    NETWORK ID     NAME              DRIVER    SCOPE
    97f34fa9fd5d   bridge            bridge    local
    97e970ad94f3   docker_gwbridge   bridge    local
    e8c6e013648a   host              host      local
    k6lk5evd8ifb   ingress           overlay   swarm
    20f0f809422d   none              null      local
    ```

----

Com isso podemos percebe uma coisa muito interessante que é que independente do Nó, seja ele gerente ou trabalhador, a rede com nome de **ingress** é a mesma em qualquer nó.

O que significa que todos as maquinas dentro do swarm tem essa rede em comum, vamos nos lembrar do routing mesh então é basicamente isso que estamos vendo, o driver overlay é o responsavel por fazer boa parte da mágica acontecer. Resolvendo todas as requisições antes de chegarem nos nós. E ela tem mais uma característica importante, é que está criptografada, garantindo segurança para o swarm.

----

## Service Discovery

O Docker Engine possui um servidor DNS embutido nele, usado por contêineres quando o Docker não está sendo executado no modo Swarm e para tarefas, quando está. Ele fornece resolução de nomes para todos os contêineres que estão no host em redes bridge, overlay. Cada contêiner encaminha suas consultas para o Docker Engine, que por sua vez verifica se o contêiner ou serviço está na mesma rede que o contêiner que enviou a solicitação em primeiro lugar. Se for, ele pesquisa o endereço IP (ou IP virtual) que corresponde a um contêiner, uma tarefa ou um nome de serviço em seu armazenamento de chave-valor interno e o retorna ao contêiner que enviou a solicitação. Bem legal, hein?

Como eu disse antes, o Docker Engine só retornará um endereço IP se o recurso correspondente estiver na mesma rede que o contêiner que gerou a solicitação. O que também é legal nisso é que os Docker Hosts armazenam apenas as entradas DNS que pertencem às redes nas quais o nó possui contêineres ou tarefas. Isso significa que eles não armazenarão informações que sejam praticamente irrelevantes para eles ou que outros contêineres não precisem saber.

Traduzindo o texto, se lembra do teste que fizemos do ping pong? Basicamente conseguimos fazer chamadas diretas pelo nome dos serviços de outros serviços usando uma rede overlay, mas não há, por padrão, e sim uma que vamos criar.

----

### Criando nossa rede overlay

Para acessar a documentação [overlay](https://docs.docker.com/network/overlay/){:target="_blank"}.

Para criar uma rede network:

```bash
docker network create -d overlay minha_rede_overlay
```

Para verificar:

=== "Gerente"

    ```bash
    docker network ls
    ```

    Saida:

    ```bash
    NETWORK ID     NAME                 DRIVER    SCOPE
    a0d052078dbe   bridge               bridge    local
    561b12e09e46   docker_gwbridge      bridge    local
    4881488b5453   host                 host      local
    k6lk5evd8ifb   ingress              overlay   swarm
    wpmvutc3phrd   minha_rede_overlay   overlay   swarm
    255b8ee96504   none                 null      local
    ```

=== "Trabalhadores"

    ```bash
    NETWORK ID     NAME              DRIVER    SCOPE
    97f34fa9fd5d   bridge            bridge    local
    97e970ad94f3   docker_gwbridge   bridge    local
    e8c6e013648a   host              host      local
    k6lk5evd8ifb   ingress           overlay   swarm
    20f0f809422d   none              null      local
    ```

----

Como podem ver, quando executamos o **docker network ls** nos gerentes conseguimos ver a rede que criamos, e nos trabalhadores não. Porque isso acontece? Bem, por de trâs dos panos ali, todos os nó já sabem da existencia dessa rede, porém os nós trabalhadores são vão listar essa rede, se um serviço usar ela explicitamente.

Vamos usar uma distro chamada alpine, que é um linux super leve que vai servi para nossa ilustração do processo.

```bash
docker service create --name teste_discovery_search --network minha_rede_overlay --replicas 2 alpine sleep 50m
```

Verifique aonde o serviço está rodando com o **docker service ps** e passe o ID do serviço:

```bash
ID             NAME                       IMAGE           NODE       DESIRED STATE   CURRENT STATE                ERROR     PORTS
wuj7ee913pop   teste_discovery_search.1   alpine:latest   manager3   Running         Running about a minute ago             
178njdwf3bjx   teste_discovery_search.1   alpine:latest   manager1   Running         Running about a minute ago
```

No meu caso, está rodando no manager3 e no manager1, para acessa o "bash" do alpine a gente precisa passar o parametro **sh**, se por acaso cair numa maquina trabalhadora, você vai conseguir visualizar a rede que criamos.

----

Vamos acessar o terminal do alpine:

=== "manager1"

    Vamos pega o nome do serviço na rede que criamo no manager1

    ```bash
    docker network inspect minha_rede_overlay
    ```

    Saida:

    ```JSON
        "Containers": {
            "c25892b0685a6a5c3ea0337c58ce694d4a6da01712988c9cdb5fbad4ef24a656": {
                "Name": "teste_discovery_search.2.178njdwf3bjx26qk81fkr8yoh",
                "EndpointID": "41c6b2419c382238425e7afc0882ecdf2ef2038003d5d80bfb7322037982e840",
                "MacAddress": "02:42:0a:00:01:03",
                "IPv4Address": "10.0.1.3/24",
                "IPv6Address": ""
            },
    ```

    Na saida JSON vai ter esse trecho, e precisamos do **Name** para conseguir realizar essa comunicação.
    No meu caso **teste_discovery_search.2.178njdwf3bjx26qk81fkr8yoh**

    Com o nome do serviço vamos usar o exec para acessar o terminal do alpine

    ```bash
    docker exec -it teste_discovery_search.2.178njdwf3bjx26qk81fkr8yoh sh
    ```

=== "manager3"

    Vamos executar o mesmo procedimento para pegar o nome do serviço no manager3

    ```bash
    docker network inspect minha_rede_overlay
    ```

    Saida:

    ```JSON
        "Containers": {
            "ccabd4e0efd2862300cbb27ffebf8ca6718ab6d0163633bdb0a388766ebe937a": {
                "Name": "teste_discovery_search.1.wuj7ee913popf8wsysemruslc",
                "EndpointID": "e518f6eeab368d6ffabd439a4b902dccd3d5996bcbabd1f79c1cb2eb56b9be42",
                "MacAddress": "02:42:0a:00:01:04",
                "IPv4Address": "10.0.1.4/24",
                "IPv6Address": ""
            },
    ```

    Na saída JSON vai ter esse trecho, e precisamos do **Name** para conseguir realizar essa comunicação.
    No meu caso **teste_discovery_search.1.wuj7ee913popf8wsysemruslc**

    Com o nome do serviço vamos usar o exec para acessar o terminal do alpine

    ```bash
    docker exec -it teste_discovery_search.1.wuj7ee913popf8wsysemruslc sh
    ```

----

Agora que estamos com o terminal de ambos os serviços, vamos usar o ping pong para realizar uma chamada.

=== "manager1"

    Use o nome do serviço do manager3

    ```bash
    ping teste_discovery_search.1.wuj7ee913popf8wsysemruslc
    ```

    Saida:

    ```bash
    PING teste_discovery_search.1.wuj7ee913popf8wsysemruslc (10.0.1.4): 56 data bytes
    64 bytes from 10.0.1.4: seq=0 ttl=64 time=0.083 ms
    64 bytes from 10.0.1.4: seq=1 ttl=64 time=0.107 ms
    64 bytes from 10.0.1.4: seq=2 ttl=64 time=0.087 ms
    64 bytes from 10.0.1.4: seq=3 ttl=64 time=0.134 ms
    64 bytes from 10.0.1.4: seq=4 ttl=64 time=0.156 ms
    64 bytes from 10.0.1.4: seq=5 ttl=64 time=0.075 ms
    ```

=== "manager3"

    Use o nome do serviço do manager1

    ```bash
    ping teste_discovery_search.2.178njdwf3bjx26qk81fkr8yoh
    ```

    Saida:

    ```bash
    PING teste_discovery_search.2.178njdwf3bjx26qk81fkr8yoh (10.0.1.3): 56 data bytes
    64 bytes from 10.0.1.3: seq=0 ttl=64 time=0.060 ms
    64 bytes from 10.0.1.3: seq=1 ttl=64 time=0.082 ms
    64 bytes from 10.0.1.3: seq=2 ttl=64 time=0.079 ms
    64 bytes from 10.0.1.3: seq=3 ttl=64 time=0.273 ms
    64 bytes from 10.0.1.3: seq=4 ttl=64 time=0.097 ms
    64 bytes from 10.0.1.3: seq=5 ttl=64 time=0.203 ms
    ```

Então nosso teste foi realizado, de máquinas diferentes, acessamos contêineres e com apenas o nome do serviço, o docker se encarregou de encontrar.
