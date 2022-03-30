## Para Instalar ✨

Vamos iniciar com a instalação do docker.

=== "Windows"

    - A documentação: [Instalação no Windows](https://docs.docker.com/desktop/windows/install/){:target="_blank"}
    - Vou deixa aqui um [passo a passo](https://www.youtube.com/watch?v=sYsIoWtS5LY){:target="_blank"} que vai ajudar nesse processo de instalação no windows.
    - Lembre-se de que o Docker para Windows possui restrições acerca de utilização e instabilidades, sendo preferível usar a versão para Linux

=== "Linux"

    - A documentação para verificar as distros: [Distros](https://docs.docker.com/engine/install/){:target="_blank"}
    - Selecione sua distribuição e continue com a instalação
    - Também é possivel usar o repositorio oficial da sua distro e opções como o Snap e Flatpak
    - non-root user: para rodar o docker no Linux, temos que chamar o sudo todas às vezes, para contornar isso aqui [documentação](https://docs.docker.com/engine/install/linux-postinstall/){:target="_blank"}
    - Ou basta usar o comando:

    ```bash
        sudo usermod -aG docker $USER
    ```

=== "Mac"

    - A documentação: [Instalação no MAC](https://docs.docker.com/desktop/mac/install/){:target="_blank"}
    - Lembre-se de que o Docker para Mac possui restrições acerca de utilização e instabilidades, sendo preferível usar a versão para Linux

----

## Hello World 🪐

Após a instalação abra seu terminal e execute:

=== "Windows"

    ```ps1
        docker run hello-world
    ```

=== "Linux"

    ```bash
        docker run hello-world
    ```

    - Se o erro do docker daemon ocorrer basta acessar a [documentação](https://docs.docker.com/config/daemon/){:target="_blank"}
    - Para rodar manualmente o backend, abra o terminal e use o comando:

    ```bash
        sudo dockerd
    ```

    - É necessário manter o terminal aberto rodando o backend.]
    - Para finalizar o processo no terminal, ctrl + c

=== "Mac"

    ```bash
        docker run hello-world
    ```

----

## Entendendo o Hello World

Se tudo ocorreu bem no hello world na saida vai ser assim:

```bash
docker run hello-world                                                                                                    126 ✘
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:97a379f4f88575512824f3b352bc03cd75e239179eea0fecc38e597b2209f49a
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Essa saida nos dá informações importantes, então vamos analisar por partes:

```bash
Unable to find image 'hello-world:latest' locally
```

Esse trecho fala sobre uma imagem que não foi encontrada localmente, o proximo trecho:

```bash
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:97a379f4f88575512824f3b352bc03cd75e239179eea0fecc38e597b2209f49a
Status: Downloaded newer image for hello-world:latest
```

Já nesse trecho o docker está baixando uma imagem de algum lugar e após isso ele executa o hello world normalmente.

----

## Afinal o que aconteceu?

Para exemplificar o docker não encontrou a imagem local e baixou de um repositório, mas qual?

??? note "Espiar a resposta"
    O repositório com as imagens é o Docker Hub.
