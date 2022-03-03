## Compose o que é?

![docker compose](https://i.postimg.cc/Vk4MbDTZ/docker-compose-orquestra.png){ loading=lazy }

O [docker compose](https://docs.docker.com/compose/){:target="_blank"} é uma ferramenta desenvolvida para ajudar a definir e compartilhar aplicativos de vários contêineres. A grande vantagem de usar o Compose é que você pode definir todos os aplicativos em um arquivo, mantê-la na raiz do repositório do seu projeto (agora com controle de versão) e permitir que outra pessoa contribua facilmente para o seu projeto.

Alguém precisaria apenas clonar seu repositório e iniciar o aplicativo de composição. Na verdade, você pode ver alguns projetos no GitHub/GitLab fazendo exatamente isso agora.

O arquivo de definição do Docker Compose é o local onde é especificado todo o ambiente (rede, volume e serviços), ele é escrito seguindo o formato YAML. Esse arquivo por padrão tem como nome docker-compose.yml.

----

## O que é o YAML?

[YAML](https://yaml.org/){:target="_blank"} é uma linguagem de serialização de dados que é frequentemente usada para gravar arquivos de configuração. Dependendo de quem você pergunta, YAML significa ainda outra linguagem de marcação ou YAML não é uma linguagem de marcação (um acrônimo recursivo), que enfatiza que YAML é para dados, não documentos.

YAML é uma linguagem de programação popular porque é legível e fácil de entender. Também pode ser usado em conjunto com outras linguagens de programação.

----

## Instalando o Docker-Compose

=== "Windows"

    + O docker compose é instalado junto ao docker

    ```ps1
    docker-compose
    ```

=== "Mac"

    + O docker compose é instalado junto ao docker

    ```bash
    docker-compose
    ```

=== "Linux"

    + No linux, basta seguir a [documentação](https://docs.docker.com/compose/install/){:target="_blank"}
    + Se houve instalação via Snap ou Flatpak, o docker compose precisa usar a mesma via
    + Após a instalação

    ```bash
    docker-compose
    ```

----

## Anatomia do docker-compose.yml

O padrão YAML utiliza a indentação como separador dos blocos de códigos das definições, por conta disso o uso da indentação é um fator muito importante, ou seja, caso não a utilize corretamente, o docker-compose falhará em sua execução.

Cada linha desse arquivo pode ser definida com uma chave valor ou uma lista. Nesse caso, vamos usar a imagem que criamos:

1. Crie uma pasta
2. Crie um arquivo chamado docker-compose.yml na pasta
3. Copiei e cole:

```YAML
version: "1"
services:
    flask_web:
        image: sposigor/app-flask-teste:1
        ports:
          - 5000:5000
```

Abra o terminal na pasta, que está o arquivo docker-compose.yml e execute:

```bash
docker-compose up -d
```

Ilustração usando o **docker-compose up**
![Saida](https://i.postimg.cc/wvk1XxTp/terminal-compose.png){ loading=lazy }

Vá em **localhost:5000** para verificar.

Ai está a mágica do compose, com um simples documento yml, podemos configurar nosso contêineres para rodar com um simples comando e para finalizar:

```bash
docker-compose down
```

Vou deixar a documentação da docker para todas as possibilidades da estrutura [docker github](https://github.com/compose-spec/compose-spec/blob/master/spec.md){:target="_blank"} é importante ler, não fique preocupado com todas as especificações, geralmente com algumas é o suficiente para subir o contêiner a partir do .yml.


[Pare de fazer confusão com o Docker Compose](https://www.youtube.com/watch?v=MGe5AdV81jo){:target="_blank"}

----

## Vamos brincar um pouco com o compose

Agora que você entendeu um pouco sobre o compose, vou deixar alguns exemplos de arquivos docker-compose.yml.

=== "MySQL"

    ```YAML
    version: "1"
    services:
      db:
        image: mysql:8.0-oracle
        restart: always
        environment:
          MYSQL_DATABASE: 'db'
          MYSQL_USER: 'user'
          MYSQL_PASSWORD: 'password'
          MYSQL_ROOT_PASSWORD: 'password'
        ports:
          - '3306:3306'
        expose:
          - '3306'
        volumes:
          - my-db:/var/lib/mysql
    volumes:
      my-db:
    ```

    + Use um editor sql de sua preferencia e faça a conexão.

=== "PostgresSQL"

    ```YAML
    version: '1'
    services:
      postgres-compose:
        image: postgres:14.-alpine
        restart: always
        environment:
          - POSTGRES_USER=postgres
          - POSTGRES_PASSWORD=postgres
        ports:
          - '5432:5432'
        volumes:
          - db:/var/lib/postgresql/data
    volumes:
      db:
        driver: local
    ```

    + Use um editor sql de sua preferencia e faça a conexão.

=== "PostgresSQL + pgadmin"

    ```YAML
    version: '1'
    services:
      postgres-compose:
        image: postgres:14.2-alpine
        environment:
          POSTGRES_USERNAME: "postgres"
          POSTGRES_PASSWORD: "postgres"
        ports:
          - "5432:5432"
        volumes:
          - db:/var/lib/postgresql/data
        networks:
          - postgres-compose-network

      pgadmin-compose:
        image: dpage/pgadmin4
        environment:
          PGADMIN_DEFAULT_EMAIL: "qualquer@email.com"
          PGADMIN_DEFAULT_PASSWORD: "postgres"
        ports:
          - "1515:80"
        depends_on:
          - postgres-compose
        networks:
          - postgres-compose-network

    volumes:
      db:
        driver: local

    networks:
      postgres-compose-network:
        driver: bridge
    ```

    + Vá no **localhost:1515**
    + login: qualquer@email.com
    + Senha: postgres
    + Pronto mais uma conexão bem sucedida

----

Além disso, recomendo verificar esse repositório da docker com múltiplos exemplos [awesome-compose](https://github.com/docker/awesome-compose){:target="_blank"}

Tente bastante, repita cada passo, crie suas variações.
