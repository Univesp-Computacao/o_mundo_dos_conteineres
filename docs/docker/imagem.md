## O que é a imagem

Pense nela como um conjunto de instruções para criar um contêiner no docker, como um modelo. A imagem do docker, contém código de aplicativo, bibliotecas, ferramentas, dependências e outros arquivos necessários para executar um aplicativo. Quando um usuário executa uma imagem, ela pode se tornar uma ou várias instâncias de um contêiner. As imagens do Docker têm várias camadas, cada uma se originando da camada anterior, mas difere dela. As camadas aceleram as compilações do Docker enquanto aumentam a capacidade de reutilização e diminuem o uso do disco. Camadas de imagem também são arquivos somente (leitura). Depois que um contêiner é criado, uma camada gravável é adicionada sobre as imagens imutáveis, permitindo que o usuário faça alterações.

Sabemos sobre o docker hub e o sobre fluxo dos contêineres, você deve estar se perguntando se é possível ter nossa própria imagem e para responder, vamos começar com o comando [docker images](https://docs.docker.com/engine/reference/commandline/images/){:target="_blank"}.

```bash
docker images
```

Vai ter um retorno das imagens disponíveis localmente

```bash
REPOSITORY                  TAG       IMAGE ID       CREATED        SIZE
ubuntu                      latest    54c9d81cbb44   4 weeks ago    72.8MB
postgres                    latest    da2cb49d7a8d   4 weeks ago    374MB
dpage/pgadmin4              latest    e52ca07eba62   7 weeks ago    272MB
hello-world                 latest    feb5d9fea6a5   5 months ago   13.3kB
dockersamples/static-site   latest    f589ccde7957   5 years ago    190MB
```

Vamos entender o cabeçalho:

+ **REPOSITORY**: É o repositório da imagem
+ **TAG**: É a tag, geralmente associada a versão
+ **IMAGE ID**: É o ID da imagem, não confunda com o ID do contêiner
+ **CREATED**: Data da criação da imagem
+ **SIZE**: Tamanho da imagem

Copie o **IMAGE ID** de uma das suas imagens locais e vamos usar o comando [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/){:target="_blank"}.

```bash
docker inspect 54c9d81cbb44
```

A saída retorna um JSON array por padrão, mostrando informações sobre a imagem, antes de continuamos, vamos observar o [docker history](https://docs.docker.com/engine/reference/commandline/history/){:target="_blank"}.

```bash
docker history 54c9d81cbb44
```

E ela vai retorna uma lista, o que é essa lista? São a camadas que quando unidas(amarrado) formam a imagem do docker que usamos. E é possível usar essas camadas em outras imagens docker, o que significa isso? Quando fazemos um pull de uma imagem, ele, na verdade, está baixando essas camadas, e o modelo(imagem) é o responsável por unificar elas para formar o resultado, então se você baixa uma imagem nova ela aproveitar as camadas que já estão disponíveis, que fazem parte do seu modelo(imagem), e baixam somente as camadas que faltam. Essa reutilização otimiza o espaço e evitar que ocorra duplicação de camadas.

## Criando nossa primeira imagem

Para continuar a explicação da criação da vamos apresentar uma nova categoria de arquivo, o [dockerfile](https://docs.docker.com/engine/reference/builder/){:target="_blank"}.

![docker file](https://www.freecodecamp.org/news/content/images/size/w1000/2021/11/docker-illustration-2.png){ loading=lazy }

Esse processo de criação de imagem é um pouco confuso, mas leia a documentação do [dockerfile](https://docs.docker.com/engine/reference/builder/){:target="_blank"} para facilitar na compreensão. Nesse processo vou subir uma aplicação flask.

1. Faça uma pasta para esse projeto.
2. Criar um arquivo chamado app.py na pasta.
3. Crie uma pasta chamada templates na pasta do projeto.
3. E por último um index.html na pasta templates.

=== "app.py"

    ```python
    from flask import Flask, render_template
    import os

    app = Flask(__name__)


    @app.route('/')
    def home():
        return render_template('index.html')


    if __name__ == "__main__":
        port = int(os.environ.get('PORT', 5000))
        app.run(debug=True, host='0.0.0.0', port=port)
    ```

=== "index.html"

    ```html
    <!DOCTYPE html>
    <html lang="">
    <head>
        <meta charset="UTF-8">
        <title>Flask no Docker</title>
    </head>
    <body>
        <h1>Eu amo Docker</h1>
    </body>
    </html>
    ```

Se quiser mudar fazer alteração no html fique a vontade.

Dentro do arquivo requirements.txt, insira:

```bash
click==8.0.3
colorama==0.4.4
Flask==2.0.2
itsdangerous==2.0.1
Jinja2==3.0.3
MarkupSafe==2.0.1
Werkzeug==2.0.2
gunicorn==20.1.0
```

Só mais uma coisa, por padrão o flask usa a porta 5000, então vamos guarda essa informação para depois.

----

### Dockerfile

Crie um arquivo chamado Dockerfile na pasta do projeto.

Insira no Dockerfile:

```dockerfile
# Iniciando a image do Python
FROM python:3.10-alpine3.15

# Copiando o requirements para o /app da imagem
COPY requirements.txt /app/requirements.txt

# Expor a porta 5000 da aplicação flask
EXPOSE 5000

# Mudando para pasta de trabalho
WORKDIR /app

# instalando as libs do requirements
RUN pip install -r requirements.txt

# Copiando o restante do conteudo no /app da imagem
COPY . /app

# Configurando o enrtypoint para rodar o python
ENTRYPOINT [ "python" ]

CMD ["app.py" ]
```

Antes de continuar com o processo, vou listar os comandos que usei no dockerfile para compreender esse processo que estamos fazendo.

```dockerfile
FROM python:3.10-alpine3.15
```

O comando FROM fala para o docker que estou buscando uma imagem, a imagem do [python](https://hub.docker.com/_/python){:target="_blank"}, e ele fornece essa estrutura do python para usamos, como base nesse processo. Lembrando que já existe milhares de imagens no docker hub.

```dockerfile
WORKDIR /app
```

Aqui estamos dizendo ao docker qual pasta usar para continuar o processo de criação da imagem. O comando WORKDIR é o local de trabalho que ele deve usar.

```dockerfile
COPY requirements.txt /app/requirements.txt
RUN pip install -r requirements.txt
```

O comando COPY, diz para copiar o conteúdo do nosso diretório, no caso o requirements.txt para o diretório da imagem e em seguida executar o pip para instalar o requirements.txt.

```dockerfile
COPY . /app
```

Continuando com a cópia, agora copiamos o restante dos arquivos em nosso diretório de trabalho local para o diretório na imagem docker.

```dockerfile
ENTRYPOINT [ "python" ]
```

**ENTRYPOINT** este é o comando que executa a aplicação no contêiner.

```dockerfile
CMD [ "app.py" ]
```

**CMD** anexa a lista de parâmetros ao parâmetro EntryPoint para executar o comando que executa o aplicativo.

Já revisamos o arquivo Dockerfile, está na hora de subir efetivamente a imagem, vamos usar um comando chamado [docker build](https://docs.docker.com/engine/reference/commandline/build/){:target="_blank"}. Abra o terminal a partir da pasta que deve está com essa estrutura:

```bash
app_web_flask
├── app.py
├── Dockerfile
├── requirements.txt
├── templates
    ├── index.html
```

Com o terminal, aberto a partir da pasta vamos executar esse comando

```bash
docker build -t univesp-github/app-flask-teste:1 .
```

Antes de executar vamos compreender, toda essa instrução, o **-t** é para nomear a imagem. O **:1** para a versão da nossa imagem e **o ponto no final** para dizer que queremos que seja feito o build a partir do diretório atual.

Para alterar o nome da imagem basta substituir o **univesp-github/app-flask-teste**.

Na execução ele roda o Dockerfiler, baixa as camadas do python e depois criar uma camada com a estrutura que criamos, para verificar, primeiro vamos dar um **docker imagens**:

```bash
REPOSITORY                       TAG                IMAGE ID       CREATED              SIZE
univesp-github/app-flask-teste   1                  0c1acf96e7e1   About a minute ago   126MB
python                           3.10-slim-buster   41471b406cc5   4 days ago           115MB
ubuntu                           latest             54c9d81cbb44   4 weeks ago          72.8MB
postgres                         latest             da2cb49d7a8d   4 weeks ago          374MB
dpage/pgadmin4                   latest             e52ca07eba62   7 weeks ago          272MB
hello-world                      latest             feb5d9fea6a5   5 months ago         13.3kB
dockersamples/static-site        latest             f589ccde7957   5 years ago          190MB
```

E lá está nossa imagem recém-criada, lembrando que a porta que o flask redireciona é a porta 5000, para manter o padrão vamos direcionar para porta 5000 da nossa maquina.

```bash
docker run -d -p 5000:5000 univesp-github/app-flask-teste:1
```

Agora no navegador, acessar a porta 5000.

```bash
localhost:5000
```

Espero que tenha entendido o processo de criação de imagem, lembrando que a documentação [dockerfile](https://docs.docker.com/engine/reference/builder/){:target="_blank"}, as variáveis que podemos inserir no Dockerfiler podem nos ajudar na construção de imagens mais completas.

----

## Subindo a imagem para o Docker Hub

Esse processo vai disponibilizar a imagem que fizemos, no repositório do docker hub.

1. Faça login em [docker hub](https://hub.docker.com/){:target="_blank"}
2. Faça login no terminal, para isso execute o comando **docker login -u** após o -u inserir o seu nome de usuário.
3. O terminal vai solicitar sua senha.

Exemplo:
```bash
docker login -u sposigor
```

Logado no terminal, vamos realizar o [docker push](https://docs.docker.com/engine/reference/commandline/push/){:target="_blank"}. Antes de continuar com o push, devemos corrigir o nome da imagem para o docker hub aceitar. Vamos usar o [docker tag](https://docs.docker.com/engine/reference/commandline/tag/){:target="_blank"}.

+ **Nome atual da imagem**: univesp-github/app-flask-teste
+ **Nome alterado da imagem**: seu usuario/app-flask-teste

Com esse ajuste, vamos subir a imagem no repositório atrelado ao usuário, se você logou no docker hub, deve ter observado o [repositório](https://hub.docker.com/repositories){:target="_blank"} da sua conta.

```bash
docker tag univesp-github/app-flask-teste:1 sposigor/app-flask-teste:1
```

Não esqueça da versão, assim que a execução finalizar, verifique as imagens [docker images](https://docs.docker.com/engine/reference/commandline/images/){:target="_blank"}, observe a alteração e vamos fazer o push dessa imagem.

```bash
docker push sposigor/app-flask-teste:1
```

Vá novamente no [repositório](https://hub.docker.com/repositories){:target="_blank"} e veja sua imagem disponível. Para visualizar o resultado do meu push [app-flask-teste](https://hub.docker.com/repository/docker/sposigor/app-flask-teste){:target="_blank"}.