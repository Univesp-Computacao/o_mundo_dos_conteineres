# O que é Kubernetes?

Para acessar o repositorio [Github Kubernetes](https://github.com/kubernetes/kubernetes){:target="_blank"} e o [Kubernetes.io](https://kubernetes.io/pt-br/){:target="_blank"}

O Google tornou Kubernetes um projeto de código-aberto em 2014. O Kubernetes combina mais de 15 anos de experiência do Google executando cargas de trabalho produtivas em escala, com as melhores idéias e práticas da comunidade.

Kubernetes é um plataforma de código aberto, portável e extensiva para o gerenciamento de cargas de trabalho e serviços distribuídos em contêineres, que facilita tanto a configuração declarativa quanto a automação. Ele possui um ecossistema grande, e de rápido crescimento. Serviços, suporte, e ferramentas para Kubernetes estão amplamente disponíveis.

![Logo Kubernete](https://cloudoki.com/content/images/2020/07/Kubernetes-Logo.wine.png){ loading=lazy }

O Kubernetes veio com grande experiência da Google e não para de crescer nas funcionalidades e usuários. É ele quem gerencia os containers em execução e por isso ele também é chamado de Orquestrador de Containers. Através dele podemos definir o estado de um sistema completo, por exemplo baseado em Microservices, seguindo boas práticas de infraestrutura como código, permitindo balanceamento de carga, alta disponibilidade, atualizações em lote e rollbacks e muito muito mais.

Hoje em dia os grandes provedores de nuvem como Azure, AWS, IBM, Red Hat, Oracle ou Google dão suporte ao Kubernetes. Além disso, existe uma implementação local chamada de Minikube que simula um cluster Kubernetes, ideal para testes e estudos. O mais legal é que as configurações locais, que definem o estado da aplicação, também rodam no Kubernetes na nuvem. Ou seja, podemos testar o orquestrador local usando Minikube e depois publicar o sistema no AWS ou Azure, apenas com poucas alterações.

----

## Por que você precisa do Kubernetes e o que ele pode fazer

Os contêineres são uma boa maneira de agrupar e executar suas aplicações. Em um ambiente de produção, você precisa gerenciar os contêineres que executam as aplicações e garantir que não haja tempo de inatividade. Por exemplo, se um contêiner cair, outro contêiner precisa ser iniciado. Não seria mais fácil se esse comportamento fosse controlado por um sistema?

É assim que o Kubernetes vem ao resgate! O Kubernetes oferece uma estrutura para executar sistemas distribuídos de forma resiliente. Ele cuida do escalonamento e do recuperação à falha de sua aplicação, fornece padrões de implantação e muito mais. Por exemplo, o Kubernetes pode gerenciar facilmente uma implantação no método canário para seu sistema.

O Kubernetes oferece a você:

+ Descoberta de serviço e balanceamento de carga O Kubernetes pode expor um contêiner usando o nome DNS ou seu próprio endereço IP. Se o tráfego para um contêiner for alto, o Kubernetes pode balancear a carga e distribuir o tráfego de rede para que a implantação seja estável.
+ Orquestração de armazenamento O Kubernetes permite que você monte automaticamente um sistema de armazenamento de sua escolha, como armazenamentos locais, provedores de nuvem pública e muito mais.
+ Lançamentos e reversões automatizadas Você pode descrever o estado desejado para seus contêineres implantados usando o Kubernetes, e ele pode alterar o estado real para o estado desejado em um ritmo controlada. Por exemplo, você pode automatizar o Kubernetes para criar novos contêineres para sua implantação, remover os contêineres existentes e adotar todos os seus recursos para o novo contêiner.
+ Empacotamento binário automático Você fornece ao Kubernetes um cluster de nós que pode ser usado para executar tarefas nos contêineres. Você informa ao Kubernetes de quanta CPU e memória (RAM) cada contêiner precisa. O Kubernetes pode encaixar contêineres em seus nós para fazer o melhor uso de seus recursos.
+ Autocorreção O Kubernetes reinicia os contêineres que falham, substitui os contêineres, elimina os contêineres que não respondem à verificação de integridade definida pelo usuário e não os anuncia aos clientes até que estejam prontos para servir.
+ Gerenciamento de configuração e de segredos O Kubernetes permite armazenar e gerenciar informações confidenciais, como senhas, tokens OAuth e chaves SSH. Você pode implantar e atualizar segredos e configuração de aplicações sem reconstruir suas imagens de contêiner e sem expor segredos em sua pilha de configuração.

----

## O que o Kubernetes não é

O Kubernetes não é um sistema PaaS (plataforma como serviço) tradicional e completo. Como o Kubernetes opera no nível do contêiner, e não no nível do hardware, ele fornece alguns recursos geralmente aplicáveis comuns às ofertas de PaaS, como implantação, escalonamento, balanceamento de carga, e permite que os usuários integrem suas soluções de logging, monitoramento e alerta. No entanto, o Kubernetes não é monolítico, e essas soluções padrão são opcionais e conectáveis. O Kubernetes fornece os blocos de construção para a construção de plataformas de desenvolvimento, mas preserva a escolha e flexibilidade do usuário onde é importante.

Kubernetes:

+ Não limita os tipos de aplicações suportadas. O Kubernetes visa oferecer suporte a uma variedade extremamente diversa de cargas de trabalho, incluindo cargas de trabalho sem estado, com estado e de processamento de dados. Se uma aplicação puder ser executada em um contêiner, ele deve ser executado perfeitamente no Kubernetes.
+ Não implanta código-fonte e não constrói sua aplicação. Os fluxos de trabalho de integração contínua, entrega e implantação (CI/CD) são determinados pelas culturas e preferências da organização, bem como pelos requisitos técnicos.
+ Não fornece serviços em nível de aplicação, tais como middleware (por exemplo, barramentos de mensagem), estruturas de processamento de dados (por exemplo, Spark), bancos de dados (por exemplo, MySQL), caches, nem sistemas de armazenamento em cluster (por exemplo, Ceph), como serviços integrados. Esses componentes podem ser executados no Kubernetes e/ou podem ser acessados por aplicações executadas no Kubernetes por meio de mecanismos portáteis, como o Open Service Broker.
+ Não dita soluções de logging, monitoramento ou alerta. Ele fornece algumas integrações como prova de conceito e mecanismos para coletar e exportar métricas.
+ Não fornece nem exige um sistema/idioma de configuração (por exemplo, Jsonnet). Ele fornece uma API declarativa que pode ser direcionada por formas arbitrárias de especificações declarativas.
+ Não fornece nem adota sistemas abrangentes de configuração de máquinas, manutenção, gerenciamento ou autocorreção.
+ Adicionalmente, o Kubernetes não é um mero sistema de orquestração. Na verdade, ele elimina a necessidade de orquestração. A definição técnica de orquestração é a execução de um fluxo de trabalho definido: primeiro faça A, depois B e depois C. Em contraste, o Kubernetes compreende um conjunto de processos de controle independentes e combináveis que conduzem continuamente o estado atual em direção ao estado desejado fornecido. Não importa como você vai de A para C. O controle centralizado também não é necessário. Isso resulta em um sistema que é mais fácil de usar e mais poderoso, robusto, resiliente e extensível.

----

## Kubernetes: como ele funciona?

A principal vantagem que as empresas garantem ao usar o Kubernetes, especialmente se estiverem otimizando o desenvolvimento de aplicações para a cloud, é que elas terão uma plataforma para programar e executar containers em clusters de máquinas físicas ou virtuais. Em termos mais abrangentes, com o Kubernetes, é mais fácil implementar e confiar totalmente em uma infraestrutura baseada em containers para os ambientes de produção. Como o propósito do Kubernetes é automatizar completamente as tarefas operacionais, ele permite que os containers realizem muitas das tarefas possibilitadas por outros sistemas de gerenciamento ou plataformas de aplicações.

O Kubernetes possibilita:

+ Orquestrar containers em vários hosts.
+ Aproveitar melhor o hardware para maximizar os recursos necessários na execução das aplicações corporativas.
+ Controlar e automatizar as implantações e atualizações de aplicações.
+ Montar e adicionar armazenamento para executar aplicações com monitoração de estado.
+ Escalar rapidamente as aplicações em containers e recursos relacionados.
+ Gerenciar serviços de forma declarativa, garantindo que as aplicações sejam executadas sempre da mesma maneira como foram implantadas.
+ Verificar a integridade e autorrecuperação das aplicações com posicionamento, reinício, replicação e escalonamento automáticos.

No entanto, o Kubernetes depende de outros projetos para oferecer plenamente esses serviços orquestrados. Com a inclusão de outros projetos open source, é possível atingir a capacidade total do Kubernetes. Dentre esses projetos necessários, incluem-se:

+ Registro, como o Atomic Registry ou o Docker Registry.
+ Rede, como o OpenvSwitch e roteamento de borda inteligente.
+ Telemetria, como o heapster, o kibana, o hawkular e o elastic.
+ Segurança, como o LDAP, o SELinux, o RBAC e o OAUTH com camadas de multilocação.
+ Automação, com a adição de playbooks do Ansible para a instalação e o gerenciamento do ciclo de vida do cluster.
+ Serviços, oferecidos em um catálogo variado de conteúdos previamente criados de padrões de aplicações populares.

----

## Como ele se encaixa na infraestrutura?

![devops](https://i.postimg.cc/L6RNMq2x/image.png){ loading=lazy }

O Kubernetes é executado em um sistema operacional e interage com pods de containers executados em nós. A máquina mestre do Kubernetes aceita os comandos de um administrador (ou equipe de DevOps) e retransmite essas instruções aos nós subservientes. Essa retransmissão é realizada em conjunto com vários serviços para automaticamente decidir qual nó é o mais adequado para a tarefa. Depois, são alocados os recursos e atribuídos os pods do nó para cumprir a tarefa solicitada.

Portanto, do ponto de vista da infraestrutura, são poucas as mudanças em comparação com a forma como você já gerencia os containers. O controle sobre os containers acontece em um nível superior, tornando-o mais refinado, sem a necessidade de microgerenciar cada container ou nó separadamente. Será necessário realizar algum trabalho, mas em sua maioria trata-se somente de uma questão de atribuir um master do Kubernetes e definir os nós e pods.

----

## Fontes usadas para elaboração desse conteúdo

[Documentação Kubernete](https://kubernetes.io/pt-br/docs/concepts/overview/what-is-kubernetes/){:target="_blank"}

[Red Hat](https://www.redhat.com/pt-br/topics/containers/what-is-kubernetes){:target="_blank"}