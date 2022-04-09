# Preste atenção

Conhecimento minimo que vai lhe ajudar a entender o processo, mas, BUT, não significa que você não possa tentar.

- Docker, conhecimento sobre container e imagens.
- Kubernetes, entender o funcionamento dos arquivos.YAML nessa atividade tão satisfatoria
- Conta no Google - Se é a primeira vez que vai acessar o [google cloud console](https://console.cloud.google.com){:target="_blank"} verificar se está no periodo de gratuidade, se não, basta criar uma nova e seguir adiante.

Com isso tudo em mãos, vamos iniciar.

## Criando o projeto no GCP

Para quem já se aventurou nessas bandas, basta criar um projeto novo para iniciar, para quem não vou deixar um passo a passo simples:

1. Com sua conta no free tier, super importante para não temos custo na nossa fatura de cartão de credito
2. Vai no [google cloud console](https://console.cloud.google.com){:target="_blank"} e verificar se está com o free tier
3. Acesse com sua conta, COM FUCK FREE TIER, e crie um projeto novo.

![imagem](https://i.postimg.cc/MTNPZFgn/versao-final-free-tier.png)

Certo?

Projeto fresquinho e free tier, tá na hora da maldade.

![gif bad](https://media3.giphy.com/media/u00B5ZZiI1d0ZdFKz8/giphy.gif?cid=ecf05e47twt2f07byfh5ywjet4y87g2xoixvtnwxmtwbarlz&rid=giphy.gif&ct=g)

## API

Vamos acessar a API necessaria para continuamos com o nosso projeto, para isso ai na barra superior AZUL, BEM NO MEIO TEM UMA LUGAR PARA PESQUISA, e digite ou copie:  **apis & services** e pronto.

Com isso, basta

![click](https://media1.giphy.com/media/XBdaS9VD83Pk63s5DM/giphy.gif?cid=ecf05e47zj8tvghc7o7rfk4o8pv2hab7sgwfm3arw6u6yaxd&rid=giphy.gif&ct=g)

EEMMMMMMMMMMMM

**Enable APIs and Services** vai está um pouco a baixo da barra azul a esquerda.

Você vai ser redirecionado para uma nova pagina bem fofinha, pesquise ISSO PESQUISE AI NA PAGINA FOFINHA o **Google Container Registry API** ele vai ter retorna o resultado e selecione o **Google Container Registry API** e você vai chegar numa pagina dessa AQUI:

![Enable](https://i.postimg.cc/J7yzzYsM/image.png)

Vai em **Enable** com o ponteiro do mouse e no mouse aperta o botão do lado esquerdo que fica geralmente no dedo indicador, se vocẽ for destro.

![tempo](https://media3.giphy.com/media/dRXSJTNTfr9AI/giphy.gif?cid=ecf05e478byn3em5eac6re4jw9lhqbz15ynnriybp6q8p0kl&rid=giphy.gif&ct=g)

Depois de alguns anos para habilitar, vai ser redirecionado lá para aquela tela de **apis & services**.

## E AGORA O QUE EU FAÇO EMMMMMM

Para ser uma pessoa democratica e levar em consideração os diferentes OS que podem passar por esse guia simpatico, vamos manter tudo no shell do GCP, BUUUUUUUUT, NÃO RECOMENDO QUE NO DIA A DIA QUE O SHELL DO GCP SEJA USADO. Naturalmente, SSH é a palavra de 3 letras mais importante para quem gosta de fuça a nuvem. Dito isso, vamos retorna.

Vá lá na BARRA AZUL no canto direito vai ter alguns ICONES, SIM ICONES, e dentre eles, vai ter ESSE AQUI OHHH:

![SHELL](https://i.postimg.cc/tgrzGKNG/image.png)

Tá vendo?

Bem, com isso vai ser aberto um terminal ai mesmo na tela na parte inferior da tela para ser mais preciso. Ele vai carregar um pouquinho e pronto, tem uma telinha no canto inferior da sua tela, porém para melhorar nossa experiencia com essa ferramenta CLASSUDA do Google, vamos aperta em **Open new window** é um icone bem ao lado do X desse terminal embutido ai.

Feito isso, ele vai abrir em uma nova aba ai do navegador que vocẽ estiver usando, agora com uma tela de respeito para trabalhar com um pouco mais de liberdade de pixel, precisamos de um projetinho para subir.

## PROJETINHO

AHHHH QUE DELICIA CARA, um projetinho fresquinho para quem gosta de um projetinho, temos aqui um projetinho feito por pessoas que gostam de um projetinho.

[Kubedoom](https://github.com/storax/kubedoom)

Perfeito, lá no projetinho o autor deixou instruções para rodar localmente, mas nosso foco é fazer a parada rodar na NUVEM.

Lá na sua PAGINA maravilhosa do Shell da Google, basta digitar:

```bash
git clone https://github.com/storax/kubedoom.git
```

E magicamente, ele vai baixar para esse local na NUVEM em seguida de um **ls** para verificar as pastas ali e TARAMMMM magicamente, HEHEHE, o nosso arquivo está ai no SHELL.

Agora, faça o SEGUINTE, precisamos entrar nas pasta pelo SHELL da NUVEM, para isso basta usa o **cd** e o nome da pasta que no NOSSO caso é **kubedoom**:

```bash
cd kubedoom
```

Agora, quero dá uma editada no arquivo ai MESMO, ISSO AI, NESSE MESMO LOCAL DA NUVEM, como faço isso?

Primeiro, veja a tela, isso a TELA, lá no canto superior direito, vai ter uma fotinha sua, ou não e olhando atentamente para esquerda vai ter alguns icones, ICONES, e um deles é uma CANETA meio deitada, que na verdade é um folk mal feito do VScode.

![Imagem](https://i.postimg.cc/qqkwJzky/image.png)

RESUMO super elaborado para lhe ajudar a compreender de forma eficiente e interativa, imaginativa, perspectativa e maravilhosativa. Agora basta ir na opção file > open > seleciona nossa pasta kubedoom para ele abrir o editor na pasta.

Certo?

Com nosso projetinho, disponivel para os proximos passos, iremos consumar o usar o docker para um teste simples com um comando simples, LEMBRANDO QUE VOCÊ PRECISA ESTÁ DENTRO DA PASTA, e com isso basta digitar no seu terminal:

