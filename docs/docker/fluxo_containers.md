## Fluxo de Criação do Contêiner

Vamos retorna para nossas perguntas

1. Porque o Contêiner não rodou que nem o hello-world
2. Se rodou o que o docker run fez então?

Para tentar responder vamos explorar mais alguns comandos do docker.

Execute a função [ps](https://docs.docker.com/engine/reference/commandline/ps/){:target="_blank"} do docker para observar a lista dos contêineres que estão rodando

```bash
docker ps
```

Mesmo após ter dado o comando docker run ubuntu, o docker ps somente retorna o cabeçalho da listagem, ou seja, não há nenhum contêiner em execução.

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Bem, se o docker ps não mostrou, tem um comando mais verboso para explicitar a listagem:

```bash
docker container ls
```

O [docker contêiner ls](https://docs.docker.com/engine/reference/commandline/container_ls/){:target="_blank"} lista os contêineres como o ps.

Porém, o resultado foi o mesmo, para resolver, devemos acrescentar o verbo -a.

=== "docker ps"

    ```bash
    docker ps -a
    ```

=== "docker container ls"

    ```bash
    docker container ls -a
    ```

Após a execução a saída, vai ser parecida com essa:

```bash
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
65adb2cb2cd5   ubuntu        "bash"     4 seconds ago    Exited (0) 4 seconds ago              competent_thompson
f690741c0bc7   ubuntu        "bash"     47 seconds ago   Exited (0) 47 seconds ago             infallible_williams
6914d474838d   hello-world   "/hello"   2 hours ago      Exited (0) 2 hours ago                upbeat_lichterman
```

Então vamos entender o significado de cada cabeçalho.

+ **CONTAINER ID**: É o ID do contêiner, identificação.
+ **IMAGE**: É a imagem do contêiner.
+ **COMMAND**: É o comando que está setado para executar
+ **CREATED**: É o tempo que passou após a criação
+ **STATUS**: É a lista de status created, restarting, running, removing, paused, exited.
+ **PORTS**: É a porta da rede que o docker está rodando, exemplo 8080
+ **NAMES**: São os nomes dos contêineres

Além disso, é possível visualizar mais opções com o comando [docker ps](https://docs.docker.com/engine/reference/commandline/ps/){:target="_blank"} e [docker container ls](https://docs.docker.com/engine/reference/commandline/container_ls/){:target="_blank"}.

----

## Voltando para as questões

Então conforme a saída do comando, o ubuntu subiu a imagem, executou o comando bash e finalizou.

Certo, que tal, alterarmos o comando padrão?

Vamos pedir aquela ajuda do --help

```bash
docker run --help
```

Agora que retornou algumas opções para o comando, temos também a sintaxe do comando que é:
> docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Interessante, nessa sintaxe está dizendo que podemos executar comandos da imagem, no caso do ubuntu, podemos usar o terminal.

```bash
docker run ubuntu sleep 2m
```

Pedir para o ubuntu rodar um sleep de 2 minutos para verificamos o status desse novo contêiner. Após enviar o comando ele travou o terminal, significa que está rodando.
Abra uma nova aba do terminal e execute um **docker ps -a** para observar a saída

```bash
CONTAINER ID   IMAGE         COMMAND      CREATED          STATUS                      PORTS     NAMES
bb1bf42f3fc3   ubuntu        "sleep 2m"   18 seconds ago   Up 17 seconds                         frosty_ishizaka
65adb2cb2cd5   ubuntu        "bash"       25 minutes ago   Exited (0) 25 minutes ago             competent_thompson
f690741c0bc7   ubuntu        "bash"       25 minutes ago   Exited (0) 25 minutes ago             infallible_williams
6914d474838d   hello-world   "/hello"     3 hours ago      Exited (0) 3 hours ago                upbeat_lichterman
```

Opá, ai está, o comando mudou, e o contêiner está com um status diferente **Up**, está indicando que está rodando.

----

## Comandos Úteis

Certo, agora entendemos um pouco do fluxo que o docker faz para rodar uma imagem. Agora vamos aprender mais alguns comandos para auxiliar nessa jornada.

Volte no terminal e execute:

```bash
docker run ubuntu sleep 5m
```

Ele vai congelar o terminal, abra uma nova aba ou terminal e execute o **docker ps** para visualizar o contêiner que está ativo e agora vamos parar esse contêiner que está em execução. Execute agora o [docker stop](https://docs.docker.com/engine/reference/commandline/stop/){:target="_blank"}, mas passe o CONTAINER ID no **seu** terminal.

```bash
docker stop 19bf7322160d
```

Após a execução do comando ele vai parar o contêiner que estava ativo, para visualizar **docker ps -a**

```bash
CONTAINER ID   IMAGE         COMMAND      CREATED          STATUS                       PORTS     NAMES
19bf7322160d   ubuntu        "sleep 5m"   41 seconds ago   Exited (137) 7 seconds ago             wizardly_shtern
bb1bf42f3fc3   ubuntu        "sleep 2m"   15 minutes ago   Exited (0) 13 minutes ago              frosty_ishizaka
65adb2cb2cd5   ubuntu        "bash"       40 minutes ago   Exited (0) 40 minutes ago              competent_thompson
f690741c0bc7   ubuntu        "bash"       40 minutes ago   Exited (0) 40 minutes ago              infallible_williams
6914d474838d   hello-world   "/hello"     3 hours ago      Exited (0) 3 hours ago                 upbeat_lichterman
```

Bem o stop para a execução, que tal tentamos o [docker start](https://docs.docker.com/engine/reference/commandline/start/){:target="_blank"}, basta passar o CONTAINER ID para iniciar o contêiner novamente:

```bash
docker start 19bf7322160d
```

Ele retorna o CONTAINER ID, basta executar o **docker ps** para validar que está rodando novamente, com isso você pode pausar com stop e reiniciar um contêiner com start. Mas, ainda não interagimos com o terminal do ubuntu, para conseguimos acessar o terminal do contêiner, vamos usar o comando [docker exec](https://docs.docker.com/engine/reference/commandline/exec/){:target="_blank"}, mas antes de executar vamos verificar o --help?

```bash
docker exec --help
```

Ele vai retorna a sintaxe e as opções, mas vamos verificar a sintaxe primeiro.
>docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Ele está nos dizendo que além das opções que nos forneceu, podemos dá um comando no contêiner e que precisamos do CONTAINER ID, vamos tentar? Veja se tem algum contêiner com a imagem do ubuntu ativa, se não, ative usando o start. Agora verifique no ps e vamos lá.

```bash
docker exec -it 19bf7322160d bash
```

O que estou falando para o docker? Que a gente quer executar um comando interativo -i e alocar um pseudo-terminal -t no CONTAINER ID  e executa o bash.

Agora no seu terminal, deve está vendo a linha de comando do contêiner que você ativou. Nesse momento você deve está vendo algo +- assim:
![Terminal](https://i.postimg.cc/tCdPNmv2/bash-ubuntu.png){ loading=lazy }

Legal né, agora temos vamos falar do [docker pause](https://docs.docker.com/engine/reference/commandline/pause/){:target="_blank"}, podemos pausa um contêiner ativo:

```bash
docker pause 19bf7322160d
```

E para sair da pausa, vamos usar o [docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/){:target="_blank"}

```bash
docker unpause 19bf7322160d
```

Que tal removemos o contêiner que está inativo? Para isso vamos usar o [docker rm](https://docs.docker.com/engine/reference/commandline/rm/){:target="_blank"}

```bash
docker rm 19bf7322160d
```

Com isso você pode fazer a remoção dos contêineres.

----

## Agora que aprendeu uns comandos novos 🙂

Por último, que tal fazemos aquele:

```bash
docker run ubuntu
```

Contudo, sabemos agora que o que acontece quando o fazemos, vamos adicionar uma opção nesse comando, vamos inserir as opções -i e -t no comando para rodar o ubuntu e observar o que acontece.

```bash
docker run -it ubuntu bash
```

E agora nosso comando run, gerou o resultado que estávamos esperando. Você está no terminal do contêiner.

## Portas

Vamos acessar uma imagem um pouco diferente agora, já vimos um hello world e um SO, vamos ver um site estático [imagem](https://hub.docker.com/r/dockersamples/static-site){:target="_blank"}, você já deve saber o que é preciso para rodar essa imagem, repita os passos:

```bash
docker pull dockersamples/static-site
```

Antes de usamos o docker run, vamos passar o comando -d, para ele não congelar no terminal enquanto usamos:

```bash
docker run -d dockersamples/static-site
```

Apos finalizar a execução do comando, faça um docker ps para observar o resultado:

```bash
CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS             NAMES
26040d6c5529   dockersamples/static-site   "/bin/sh -c 'cd /usr…"   9 seconds ago   Up 8 seconds   80/tcp, 443/tcp   nervous_goldberg
```

Perceba que temos agora uma informação em **PORTS**, ele está dizendo que está disponível na porta 80, vamos verificar?

No seu navegador:

```bash
localhost:80
```

O navegador retorna um erro, vamos tentar a porta 443 e o resultado é o mesmo, porque não conseguimos ver?
Por causa da estrutura de isolamento dos contêineres, não devemos esquecer que o isolamento está presente.
Para resolver isso, remover o contêiner que acabamos de criar e passa a opção --force.

```bash
docker rm 26040d6c5529 --force
```

Após a execução, vamos passa mais uma opção para o comando run o -P que vai explicitar as portas e redirecionar automaticamente as portas do contêiner para as portas da nossa maquina:

```bash
docker run -d -P dockersamples/static-site
```

Use o docker ps para observar a saída, e em **PORTS** a saída está um pouco diferente, para visualizamos com mais clareza, vamos usar o [docker port](https://docs.docker.com/engine/reference/commandline/port/){:target="_blank"}.

```bash
docker port ca557128fd50
```

E a saída vai ser ± assim:

```bash
443/tcp -> 0.0.0.0:49153
443/tcp -> :::49153
80/tcp -> 0.0.0.0:49154
80/tcp -> :::49154
```

A porta 443 foi redirecionada para a porta 49153 do nosso sistema e a 80 para 49154, por padrão a porta 443 e 80 são sempre liberadas no contêineres, vamos verificar?

No navegador:

```bash
localhost:49154
```

E a resposta do navegador vai ser essa aqui:
![Docker Site](https://i.postimg.cc/m2vXMNYS/docker-site.png){ loading=lazy }

Além do -P também tem o -p, que permite que façamos o mapeamento específico da porta, vamos tentar?

Antes vou remover o contêiner atual:

```bash
docker rm ca557128fd50 --force
```

E vamos usar o comando:

```bash
docker -d -p 8080:80 dockersamples/static-site
```

Explicando o que fiz, passei a porta 8080 da minha maquina, para refletir a porta 80 do contêiner.
