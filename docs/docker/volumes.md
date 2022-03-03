# Gerenciamento de Dados

As referências ao espaço em disco em imagens e contêineres do Docker podem ser confusas. É importante distinguir entre armazenamento local e armazenamento virtual. Armazenamento local refere-se ao espaço em disco que a camada gravável de um contêiner usa, enquanto o armazenamento virtual é o espaço em disco usado para o contêiner e a camada gravável. A camada somente leitura de uma imagem podem ser compartilhadas entre qualquer contêiner iniciado a partir da mesma imagem.

Que tal verificar essa informação? Antes de continuamos, vou limpar minhas imagens e meus contêineres que estão armazenados, como pode ver:

=== "Imagens"

    ```bash
    REPOSITORY                       TAG                IMAGE ID       CREATED        SIZE
    sposigor/app-flask-teste         1                  c704ddb7c517   2 hours ago    60.3MB
    univesp-github/app-flask-teste   1                  c704ddb7c517   2 hours ago    60.3MB
    python                           3.10-alpine        a0e2910f7263   4 days ago     48.6MB
    python                           3.10-slim-buster   41471b406cc5   4 days ago     115MB
    python                           3.10               e4bdbb3cacf1   4 days ago     917MB
    ubuntu                           latest             54c9d81cbb44   4 weeks ago    72.8MB
    postgres                         latest             da2cb49d7a8d   4 weeks ago    374MB
    dpage/pgadmin4                   latest             e52ca07eba62   7 weeks ago    272MB
    hello-world                      latest             feb5d9fea6a5   5 months ago   13.3kB
    dockersamples/static-site        latest             f589ccde7957   5 years ago    190MB
    ```

=== "Contêineres"

    ```bash
    CONTAINER ID   IMAGE                              COMMAND                  CREATED        STATUS                    PORTS     NAMES
    46cab0670fc8   univesp-github/app-flask-teste:1   "python app.py"          2 hours ago    Exited (0) 2 hours ago              elegant_brown
    b50c4ac105c3   f32698e186b5                       "python app.py"          2 hours ago    Exited (0) 2 hours ago              boring_knuth
    40ea6243bbc3   64160aa2a8dc                       "python app.py"          2 hours ago    Exited (0) 2 hours ago              unruffled_roentgen
    7ef49050e8df   1647b4c47b68                       "python app.py"          2 hours ago    Exited (0) 2 hours ago              nostalgic_dirac
    19bf7322160d   ubuntu                             "sleep 5m"               20 hours ago   Exited (0) 19 hours ago             wizardly_shtern
    bb1bf42f3fc3   ubuntu                             "sleep 2m"               20 hours ago   Exited (0) 20 hours ago             frosty_ishizaka
    65adb2cb2cd5   ubuntu                             "bash"                   20 hours ago   Exited (0) 20 hours ago             competent_thompson
    f690741c0bc7   ubuntu                             "bash"                   20 hours ago   Exited (0) 20 hours ago             infallible_williams
    6914d474838d   hello-world                        "/hello"                 23 hours ago   Exited (0) 20 hours ago             upbeat_lichterman
    ```

Vou usar um comando um pouco diferente o [docker system prune](https://docs.docker.com/engine/reference/commandline/system_prune/){:target="_blank"}

```bash
docker system prune -af
```

Ele remove todas as imagens, contêineres, volumes e o que mais tiver armazenado no docker, passei o parâmetro -a de all e -f de force. Vou lista mais alguns comandos de remoção que podem ser uteis:

+ [docker container rm](https://docs.docker.com/engine/reference/commandline/container_rm/){:target="_blank"}: Remove somente com o CONTAINER ID.
+ [docker container prune](https://docs.docker.com/engine/reference/commandline/container_prune/){:target="_blank"}: Remove todos os containeres
+ [docker image rm](https://docs.docker.com/engine/reference/commandline/image_rm/){:target="_blank"}: Remove somente a imagem com o IMAGE ID
+ [docker image prune](https://docs.docker.com/engine/reference/commandline/image_prune/){:target="_blank"}: Remove todas as imagens
+ [docker volume rm](https://docs.docker.com/engine/reference/commandline/volume_rm/){:target="_blank"}: Remove o volume com o VOLUME NAME
+ [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/){:target="_blank"}: Remove todos os volumes
+ [docker network rm](https://docs.docker.com/engine/reference/commandline/network_rm/){:target="_blank"}: Remove o network com o NETWORK ID
+ [docker network prune](https://docs.docker.com/engine/reference/commandline/network_prune/){:target="_blank"}: Remove todos os network

----

Para começar a entender melhor o volume, armazenamento local e armazenamento virtual, recorremos ao nosso querido ubuntu, dessa vez, vamos chamar ele duas vezes:

=== "Primeira"

    ```bash
    docker run -it ubuntu bash
    ```

    + Assim que subir o bash execute o **apt-get update**
    + Escreva exit para finalizar

=== "Segunda"

    ```bash
    docker run -it ubuntu bash
    ```

    + Nesse caso apenas e escreva exit para finalizar

Com o comando **docker ps -as** a saída:

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                            PORTS     NAMES            SIZE
a4928a7ff9a6   ubuntu    "bash"    47 seconds ago   Exited (0) 44 seconds ago                   funny_thompson   5B (virtual 72.8MB)
0516c2a0ac00   ubuntu    "bash"    4 minutes ago    Exited (130) About a minute ago             fervent_turing   34MB (virtual 107MB)
```

Temos uma nova coluna que é a **SIZE** e ela possui duas informações o armazenamento local e armazenamento virtual, usando o ubuntu como exemplo, em um apenas subimos a imagem sendo o primeiro na lista e o segundo fizemos o apt-get update.

+ **Armazenamento Local**: É a camada da qual temos acesso a gravar.
+ **Armazenamento Virtual**: É o tamanho da imagem sem qualquer gravação.

Quando fazemos gravações, esses dados reflete tanto no armazenamento local quanto no armazenamento virtual.

![Ilustração Docker](https://i.postimg.cc/hPYjHztJ/ilustracao.png){ loading=lazy }

----

## Persistência de dados

Podemos querer que os dados da nossa aplicação sejam persistentes, porque assim garantimos que ela esteja distribuída e disponível se precisarmos consultá-la. Porém, se escrevermos os dados nos contêineres, por padrão eles não ficarão armazenados nesta camada, criada para ser descartável. Existem três possibilidades para contornar esta situação com o Docker.

![Persistencia dos dados](https://docs.docker.com/storage/images/types-of-mounts-volume.png){ loading=lazy }

----

### Bind Mounts

Os bind mounts estão disponíveis no Docker desde seus primeiros dias para a persistência de dados. Os bind mounts montarão um arquivo ou diretório em seu contêiner a partir de sua máquina host, que você pode referenciar através de seu caminho absoluto.

Para usar o bind mounts, o arquivo ou diretório não precisa já existir em seu host do Docker. Se não existir, será criado sob demanda. As montagens de ligação dependem do sistema de arquivos da máquina host ter uma estrutura de diretório específica disponível. Você deve criar explicitamente um caminho para o arquivo ou pasta para colocar o armazenamento.

Outra informação importante sobre bind mounts é que elas dão acesso a arquivos confidenciais. Conforme os documentos do Docker, você pode alterar o sistema de arquivos do host através de processos executados em um contêiner. Isso inclui criar, modificar e excluir arquivos e diretórios do sistema, que podem ter implicações de segurança bastante graves. Pode até impactar processos não-Docker.

Para começar a usar o bind mounts, precisamos criar uma pasta na nossa maquina, vou cria na área de trabalho e nomeá-la de bind. Copie a localização que vamos usar.

=== "-v"

    ```bash
    docker run -v /home/sposigor/bind:/teste -it ubuntu bash
    ```

=== "--mount"

    ```bash
    docker run -it --mount type=bind,src=/home/sposigor/bind,dst=/teste ubuntu bash
    ```

A docker recomenda usar **--mount** e não o **-v**

Se tiver dúvidas, na documentação do [docker run](https://docs.docker.com/engine/reference/commandline/run/){:target="_blank"} tem mais informações.

Para realizar o teste:

1. Faça o docker run, seja em -v ou --mount.
2. No bash do contêiner, de um **ls**
3. Veja a pasta **teste** criada e vamos entrar nela com o **cd teste/**
4. Na pasta **touch teste1.txt** e escreva exit para sair
5. Vá na sua pasta que você criou na máquina local e abra ela e o arquivo **teste1.txt** vai está lá.
6. Repita o processo de 1 á 3, com um novo contêiner e o arquivo criado no contêiner anterior persiste lá.

![Teste bind mount](https://i.postimg.cc/mgkPxrKD/bind-mount-teste.png){ loading=lazy }

----
### Volumes

Os volumes são um ótimo mecanismo para adicionar uma camada de persistência de dados em seus contêineres do Docker, especialmente para uma situação em que você precisa persistir dados após desligar seus contêineres.

Volumes, portanto é a solução mais recomendada pelo docker para usar em ambientes de produção e afins.

O [docker volume ls](https://docs.docker.com/engine/reference/commandline/volume_ls/){:target="_blank"} observa os volumes disponíveis:

```bash
DRIVER    VOLUME NAME
```

+ **DRIVER**: É o volume do drive
+ **VOLUME NAME**: É o nome do volume

Criar um volume novo [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/){:target="_blank"} e o nome do volume

```bash
docker volume create testando
```

Vamos colocar em produção:

=== "-v"

    ```bash
    docker run -v testando:/teste -it ubuntu bash
    ```

=== "--mount"

    ```bash
    docker run -t -i --mount type=bind,src=testando,dst=/teste ubuntu bash
    ```

1. Faça o docker run, seja em -v ou --mount.
2. No bash do container, de um **ls**
3. Veja a pasta **teste** que foi criada e vamos entrar nela com o **cd teste/**
4. Dentro da pasta **touch teste1.txt** e escreva exit para sair
6. Repita o processo de 1 á 3, com um novo container e o arquivo criado no container anterios persiste lá.

Deve ter surgido uma duvida, aonde está esse arquivo que criei?

Para isso vamos para o terminal:

1. No terminal **sudo su**
2. **cd /var/lib/docker/volumes**
3. **ls** para verificar os arquivos
4. O volume criado vai está aqui, **cd testando**
5. **ls** vai ter uma pasta **_data**
6. **cd _data** e em seguida **ls**
7. Encontrou o arquivo que foi criado

Os volumes do Docker são completamente manipulados pelo próprio Docker e, portanto, independentes da estrutura de diretórios e do sistema operacional da máquina host. Quando você usa um volume, um novo diretório é criado no diretório de armazenamento do Docker na máquina host e o Docker gerencia o conteúdo desse diretório.

Por ultimo podemos passar diretamente do [docker run](https://docs.docker.com/engine/reference/commandline/run/){:target="_blank"} o volume que ele se encarrega de criar

=== "-v"

    ```bash
    docker run -v volume_novo:/teste -it ubuntu bash
    ```

=== "--mount"

    ```bash
    docker run -t -i --mount type=bind,src=volume_novo,dst=/teste ubuntu bash
    ```

[docker volume ls](https://docs.docker.com/engine/reference/commandline/volume_ls/){:target="_blank"} e a saída:

```bash
DRIVER    VOLUME NAME
local     testando
local     volume_novo
```

----

### tmpfs mounts

O [tmpfs mounts](https://docs.docker.com/storage/tmpfs/){:target="_blank"} é para espaços de armazenamento pequenos e efêmeros que só podem ser usados ​​por um único contêiner, existe o sistema de arquivos tmpfs. Ele é apoiado apenas pelo armazenamento RAM no sistema host.

Essa funcionalidade só está disponível se você estiver executando o Docker no Linux.

Seguindo a ordem para criar um tmpfs é mais simples, já possui a própria flag --tmpfs:

```bash
docker run -it --tmpfs=/app ubuntu bash
```
![tmpfs exemplo](https://i.postimg.cc/PJXn922w/tmpfs.png){ loading=lazy }

Perceba que a pasta app que criamos usando o --tmpfs, está com um fundo verde, assim como o tmp, então significa que ela é temporária, quando o contêiner for finalizado, os dados ali são apagados.

Então qual é a utilidade de usar o tmpfs? É que ela armazena os arquivos na memória do host e não na camada de escrita do contêineres r/w, ou seja, se existem dados sensíveis para serem armazenados no processo e por questões de segura não deseja escrever na camada de escrita do contêiner r/w, o tmpfs é recomendando nessa situação.

Para mais uma aplicação podemos usar como parâmetro do --mount:

```bash
docker run -it --mount type=tmpfs,dst=/teste ubuntu bash
```