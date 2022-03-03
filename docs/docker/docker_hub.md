## O que é o Docker Hub? 🤔

Para exemplificar, o Docker Hub é um repositório que possui imagens para o docker

Para acessar [Docker Hub](https://hub.docker.com/){:target="_blank"}

1. Na página inicial, pesquisaremos o hello-world.
2. Pesquise e chegaremos nessa página [hello-world](https://hub.docker.com/_/hello-world){:target="_blank"}
3. Lá encontraremos uma referência e informações importantes
4. Também repare que existe uma opção de fazer um pull, ou seja, baixa a imagem localmente


Existe um selo oficial das imagens que estão no docker hub:
![Imagem VMxContainers](https://i.postimg.cc/zXB37YFz/docker-hello-world.png){ loading=lazy }

Quando for necessário pesquise a imagem.

----

## Ubuntu: um SO ⭐

Realizaremos um exercício

+ Faça o pull da imagem do [Ubuntu](https://hub.docker.com/_/ubuntu){:target="_blank"}

```bash
    docker pull ubuntu
```

+ Rode a imagem no docker

```bash
    docker run ubuntu
```

Após ter realizado a etapa 1 e 2, porque a interface do Ubuntu não apareceu?

Agora temos mais perguntas que respostas.

1. Porque o Contêiner não rodou que nem o hello-world
2. Se rodou o que o docker run fez então?
