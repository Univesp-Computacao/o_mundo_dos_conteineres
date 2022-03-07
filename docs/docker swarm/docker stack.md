# Docker Stack

A Docker Stack é um comando do Docker para gerenciar as pilhas do Docker. Podemos usar este comando para implantar uma nova pilha ou atualizar uma existente, listar pilhas, listar as tarefas na pilha, remover uma ou mais pilhas e listar os serviços na pilha. Devemos habilitar o modo swarm para executar este comando, pois só podemos implantar pilhas no modo swarm do Docker e ele está incluído no mecanismo do Docker, para não precisarmos instalar nenhum pacote adicional, apenas precisamos habilitá-lo, pois está desabilitado por predefinição.

A proposta aqui é roda uma infraestrutura composto por vários serviços e usando o docker swarm.

No docker hub. tem uma imagem bem popular para exemplificar todo esse processo, [aqui](https://github.com/dockersamples/example-voting-app){:target="_blank"} você acessa o material referente a essa aplicação.

Antes de continuamos, vou reiniciar a página do docker play e selecionar a configuração de 3 manager and 2 workes.

Vamos criar um docker-compose.yml:

Copie o Conteúdo abaixo:

```YAML
version: "3"
services:

  redis:
    image: redis:alpine
    networks:
      - frontend
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
        
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
    environment:
        POSTGRES_HOST_AUTH_METHOD: trust

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    depends_on:
      - db
      - redis
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
      placement:
        constraints: [node.role == worker]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - 8080:8080
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

Vá no terminal e digite:

```bash
cat > docker-compose.yml
```

De um enter, depois um shift + insert para colar o conteudo copiado.

De um enter para abrir um espaço no final e ctrl + d, para sair da edição do arquivo.

Agora com o .yml certinho, vamos usar o [docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/){:target="_blank"} para subir toda essa aplicação.

=== "Execute"

    ```bash
    docker stack deploy --compose-file docker-compose.yml vote
    ```

=== "Saida"

    ```bash
    Creating network vote_backend
    Creating network vote_frontend
    Creating network vote_default
    Creating service vote_db
    Creating service vote_vote
    Creating service vote_result
    Creating service vote_worker
    Creating service vote_visualizer
    Creating service vote_redis
    ```

----

Vamos verificar agora com o [docker stack ls](https://docs.docker.com/engine/reference/commandline/stack_ls/){:target="_blank"}.

=== "docker stack ls"

    ```bash
    docker stack ls
    ```

    Saida:

    ```bash
    NAME      SERVICES   ORCHESTRATOR
    vote      6          Swarm
    ```

=== "docker service ls"

    ```bash
    docker service ls
    ```

    Saida

    ```bash
    ID             NAME              MODE         REPLICAS   IMAGE                                          PORTS
    a71bfc34hnw6   vote_db           replicated   1/1        postgres:9.4                                   
    3c11wqgvyvhe   vote_redis        replicated   1/1        redis:alpine                                   
    5bghvabs9joq   vote_result       replicated   1/1        dockersamples/examplevotingapp_result:before   *:5001->80/tcp
    5pn9qep3hqji   vote_visualizer   replicated   1/1        dockersamples/visualizer:stable                *:8080->8080/tcp
    u8pgdms926le   vote_vote         replicated   2/2        dockersamples/examplevotingapp_vote:before     *:5000->80/tcp
    vwlu9rb7yycs   vote_worker       replicated   1/1        dockersamples/examplevotingapp_worker:latest
    ```

----

E com isso subimos completamente a aplicação, se por acaso você executar muito rápido as instruções, talvez veja as réplicas subindo.

Com isso vamos abrir as portas para verificar o resultado.

+ **8080**: Para visualizar o dashboard
+ **5000**: Para interagir com a votação
+ **5001**: Para visualizar o resultado da votação

No dashboard é possível ver quais nós, estão rodando os contêineres.

Com isso finalizamos esse conteúdo sobre docker swarm, espero que tenha gostado e aprendido.
