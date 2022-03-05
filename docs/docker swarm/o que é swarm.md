# O que é o Swarm?

Imagine uma orquestra, com todas aquelas pessoas tocando instrumentos e com um maestro que transforma, direcionar e guia o ritmo da música.

[Exemplo de Orquestra](https://www.youtube.com/watch?v=An7Y321fG7g){:target="_blank"}

E em seguida transformamos isso em código e estrutura, chegamos nesse modelo:

+ **Pessoas que tocam a orquestra**: São os computadores, para cada pessoa, imagine um computador
+ **Instrumentos**: Para cada instrumento, imagine que seja um docker/trabalhador
+ **Maestro**: Esse é o cara, no docker, chamamos ele de Swarm/gerente, é o responsável por dividir, gerenciar e garantir a estabilidade e tudo isso sem precisar que alguém fique de olho.
+ **Musica**: Por ultimo a musica, qual nota tocar? quando tocar? atento a cada nota, se houver um erro é resolvido por ele e essa vamos chamar conta ineres, que vão ser controlados pelo gerente.

Ilustração simples:

![Cluster](https://i.postimg.cc/QtZ061JB/docker-cluster.png){ loading=lazy }

----

## Um pouco de teoria

As características de gerenciamento de agrupamento e orquestração incorporadas no Docker Engine são construídas utilizando o swarmkit. O Swarmkit é um projeto separado que implementa a camada de orquestração do Docker sendo utilizado diretamente dentro do Docker.

Um swarm consiste em múltiplos hospedeiros Docker que funcionam em modo swarm e agem como gerentes (para gerir membros e delegação) e trabalhadores (que gerem serviços de swarm). Um determinado anfitrião Docker pode ser um gerente, um trabalhador, ou desempenhar ambas as funções. Quando se cria um serviço, define-se o seu estado ideal (número de réplicas, rede e recursos de armazenamento disponíveis, portas que o serviço expõe ao mundo exterior, e mais). O Docker trabalha para manter esse estado desejado. Por exemplo, se um nó de trabalhador ficar indisponível, Docker programa as tarefas desse nó em outros nós. Uma tarefa é um contêiner em funcionamento que faz parte de um serviço de swarm sendo gerido por um gerente de swarm, em oposição a um contêiner independente.

Uma das principais vantagens dos serviços de swarm sobre os contêineres independentes é que se pode modificar a configuração de um serviço, incluindo as redes e os volumes a que está ligado, sem necessidade de reiniciar manualmente o serviço. O Docker irá atualizar a configuração, parar as tarefas do serviço com a configuração desatualizada, e criar que correspondam à configuração desejada.

Quando o Docker funciona em modo de swarm, ainda é possível executar contêineres independentes em qualquer um dos anfitriões Docker que participam no swarm, bem como serviços de swarm. Uma diferença chave entre contêineres independentes e serviços de swarm é que apenas os gerentes de swarm podem gerir um swarm, enquanto os contêineres independentes podem ser iniciados em qualquer daemon. O Docker daemons da doca podem participar num swarm como gerentes, trabalhadores, ou ambos.

Da mesma forma que se pode usar Docker Compose para definir e gerir contêineres, pode-se definir e gerir pilhas de serviços de Swarm.

Continue a ler para obter detalhes sobre conceitos relacionados com os serviços de swarm de Docker, incluindo nós, serviços, tarefas, e equilíbrio de carga mais conhecido como Load balancing.

A documentação docker, vai definir o swarm/gerentes como manage e os computadores com o docker de works e os noś de nodes.

----

## Nodes - nó

Um nó é um exemplo do motor Docker que participa no swarm. Também se pode pensar nisto como um nó de Docker. Pode-se correr um ou mais nós num único computador físico, ou em um servidor de nuvem, mas as implementações de swarm de produção incluem tipicamente nós Docker distribuídos por múltiplas máquinas físicas e de nuvem.

Para implantar a sua aplicação num swarm, submete uma definição de serviço a um nó gerente. O nó gerente despacha unidades de trabalho chamadas tarefas para nós de trabalhadores.

Os nós gerentes também executam as funções de orquestração e gerenciamento de agrupamento necessárias para manter o estado desejado do swarm. Os nós gerentes elegem um único líder para realizar tarefas de orquestração.

Esses nós trabalhadores recebem e executam as tarefas enviadas pelos nós gerentes. Por defeito, os nós gerentes também executam serviços como nós trabalhadores, mas pode configurá-los para executar tarefas de gerenciamento exclusivamente e ser nós apenas gerentes. Um agente corre em cada nó de trabalhadores e informa sobre as tarefas que lhe são atribuídas. O nó trabalhadores notifica o nó gerente do estado atual das suas tarefas atribuídas, para que o gerente possa manter o estado desejado de cada trabalhadores.

----

## Tarefas e serviços

Um serviço é a definição das tarefas a executar nos nós do gerente ou do trabalhador. É a estrutura central do sistema do swarm e a raiz primária da interação do usuário com o swarm.

Quando se cria um serviço, especifica-se a imagem do contêiner a utilizar e os comandos a executar dentro de contêineres em funcionamento.

No modelo de serviços replicados, o gerente do swarm distribui um número específico de tarefas replicadas entre os nós, com base na escala que se define no estado desejado.

Para serviços globais, o swarm executa uma tarefa para o serviço em cada nó disponível no agrupamento.

Uma tarefa transporta um contêiner Docker e os comandos para correr dentro do contêiner. É a unidade de programação atômica do swarm. Os nós gerentes atribuem tarefas aos nós de trabalhadores conforme o número de réplicas definidas na escala de serviço. Uma vez atribuída uma tarefa a um nó, este não pode deslocar-se para outro nó. Só pode funcionar com o nó atribuído ou falhar.

----

## Load balancing

O gerente do swarm utiliza o equilíbrio da carga de entrada para expor os serviços que pretende disponibilizar externamente ao swarm. O gerente de swarm pode atribuir automaticamente ao serviço em uma porta ou pode configurar uma porta para o serviço. É possível especificar qualquer porta não utilizada. Se não especificar uma porta, o gerente do swarm atribui ao serviço uma porta no intervalo 30000-32767.

Componentes externos, tais como equilibradores de carga de nuvem, podem acessar o serviço na porta de qualquer nó do cluster, quer o nó esteja ou não a executar a tarefa para o serviço. Todos os nós na rota de entrada do swarm de ligações a uma instância de tarefa em execução.

O modo swarm tem um componente DNS interno que atribui automaticamente a cada serviço no swarm uma entrada DNS. O gerente do swarm utiliza o equilíbrio de carga interno para distribuir pedidos entre serviços dentro do cluster, com base no nome DNS do serviço.

----

![Raft](https://i.postimg.cc/gkBsw-qW3/raft-guia.png){ loading=lazy }

## Raft - Consenso Distribuídos

Antes de continuamos a estudar sobre swarm é importante falar sobre um algoritmo que se chama Raft, o responsável pelas decisões, é o cérebro dessa estrutura, para isso vou deixa aqui a [Raft](https://raft.github.io/){:target="_blank"}.

É muito importante que você entenda um pouco sobre o Raft, já que essa expressão vai se repetir mais algumas vezes, em kubernetes, por exemplo. Entender o raft é criar melhores aplicações que usam docker, kubernetes e outras soluções que usam esse cérebro por trás das decisões.

Nesse ponto, encontrei dois vídeos didáticos sobre o assunto.

[Linuxtips](https://www.youtube.com/watch?v=N6G4bv-qo4w) usar a explicação visual do [Ben Johnson](http://thesecretlivesofdata.com/raft/) para explicar sobre raft

E também [Algoritmos de consenso em sistemas distribuídos (teoria e prática) - Edward Ribeiro](https://www.infoq.com/br/presentations/algoritimos-consenso-sistemas-distribuidos/) que vai abordar o raft e mais alguns algoritmos de consenso.
