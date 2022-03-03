# Falaremos sobre Contêineres?

Para iniciar essa conversa acho importante ter uma visão geral sobre o conteúdo é importante que você assista o video do [Código Fonte TV](https://www.youtube.com/watch?v=-pUZBovqRcU){:target="_blank"} que explica sobre contêineres.

## Como contêineres funcionam?

Temos agora uma base teórica para trabalhar, reforçaremos o conceito de Máquina Virtual e Contêineres:

+ **Contêineres** são medidos em megabyte. Eles contêm, no máximo, a aplicação e os arquivos necessários para executá-la. Além disso, eles costumam ser usados para empacotar funções individuais que realizam tarefas específicas, os famosos microsserviços. Como são leves e têm um sistema operacional compartilhado, os contêineres são muito fáceis de migrar entre vários ambientes.
+ **As máquinas virtuais(VM)** são medidas em gigabyte. Eles costumam ter seu próprio sistema operacional, possibilitando a execução simultânea de várias funções com uso intenso de recursos. Por terem um número maior de recursos à disposição, as máquinas virtuais conseguem abstrair, dividir, duplicar e emular por inteiro servidores, sistemas operacionais, desktops, bancos de dados e redes.

----
Temos essa imagem para ilustra melhor esse conceito:
![Imagem VMxContainers](https://i.postimg.cc/PrxmYTnQ/Virtualizacaox-Container.png){ loading=lazy }

+ **Virtualização**: O hipervisor é um software que separa os recursos das respectivas máquinas físicas para eles poderem ser particionados e dedicados às máquinas virtuais. Quando o usuário emite uma instrução de máquina virtual que exige mais recursos do ambiente físico, o hipervisor retransmite a solicitação ao sistema físico e armazena as mudanças em memória transitória. As máquinas virtuais são similares aos servidores físicos e agem como eles, o que pode multiplicar as desvantagens de grandes infraestruturas de sistema operacional e das dependências da aplicação. Na maioria das vezes, essas infraestruturas não são necessárias para executar uma aplicação ou microsserviços.
+ **Contêineres**: Os contêineres armazenam um microsserviço ou aplicação, além de todos os elementos necessários para executá-los. Tudo que eles contêm é mantido em um recurso chamado imagem: um arquivo baseado em código que inclui todas as bibliotecas e dependências. Pense nesses arquivos como uma instalação da distribuição Linux, já que a imagem inclui pacotes RPM e arquivos de configuração. Como os contêineres são muito pequenos, geralmente há centenas deles levemente acoplados.

----
Já que sabemos a diferença entre VM e Contêineres, precisamos entender aonde essa solução se encaixa, pois, com as mudanças que a tecnologia traz no decorrer da sua história, é preciso entender as novas práticas de TI com as tradicionais para chegamos a um ponto-chave dessa pergunta:

+ **As novas práticas de TI** (desenvolvimento nativo em nuvem, CI/CD e DevOps) existem graças à divisão das cargas de trabalho nas menores unidades úteis possíveis, que geralmente são uma função ou um microsserviço. Essas unidades são melhor empacotadas em contêineres. Assim, várias equipes podem trabalhar em partes separadas de uma aplicação ou serviço sem interromper, ou pôr em risco o código empacotado em outros contêineres.
+ **Nas arquiteturas de TI tradicionais** (monolíticas e legadas), todos os elementos de uma carga de trabalho são mantidos em um arquivo grande que não pode ser dividido. Por isso, ele precisa ser empacotado como uma unidade completa em um ambiente maior, frequentemente uma máquina virtual. Era comum criar e executar uma aplicação inteira dentro de uma máquina virtual, mesmo sabendo que ao armazenar todo o código e dependências, ela ficava grande demais, e isso poderia gerar falhas em cascata e downtime durante as atualizações.

----

## Se aprofundando um pouco mais nos Contêineres

Se em algum momento você já usou uma VM, imagino que tenha passado pelo processo de baixa uma SO, seja Linux ou Windows e alocando os recursos conforme a capacidade do hardware disponível, então esse processo dos Contêineres pode parecer meio confuso, para isso deixei duas perguntas que precisamos responder antes de contínua.

+ Se não é uma virtualização como ocorre o isolamento?
+ Como ocorre a divisão de recursos?

### Se não é uma virtualização como ocorre o isolamento?

Existe uma palavra importante, que resolve essa questão **Namespaces**, permitindo que os contêineres consigam isolamento em determinados níveis que são:

+ **PID**: Isola os processos rodando dentro do contêiner
+ **NET**: Isola as interfaces de redes
+ **IPC**: Isola a comunicação entre processos e memoria
+ **MNT**: Isola o ponto de montagem
+ **UTS**: Isola o Kernel, ou seja, simula um novo host(usuário)
+ **USER**: Isola os arquivos do sistema

Dentre esse processo o mais importante, é o UTS, o responsável pelo isolamento dos contêineres, ao simular um novo host, ele usar o Kernel da máquina física para o processamento, porém isoladamente.

Os namespaces fazem parte do kernel do Linux desde 2002, os namespaces são responsáveis por gerar o isolamento de grupos de processos em seu nível lógico.

### Como ocorre a divisão de recursos?

O gerenciamento de recursos, tem a palavra **Cgroups** para resolver essa demanda de gerenciar, seja ela automaticamente ou manualmente. Os cgroups fornecem os recursos:

+ **Limites de recursos**: você pode configurar um cgroup para limitar quanto de um determinado recurso (memória ou CPU, por exemplo) um processo pode usar.
+ **Priorização**: Você pode controlar quanto de um recurso (CPU, disco ou rede) um processo pode usar em comparação com processos em outro cgroup quando há contenção de recursos.
+ **Contabilidade**: os limites de recursos são monitorados e relatados no nível do cgroup.
+ **Controle**: Você pode alterar o status (congelado, interrompido ou reiniciado) de todos os processos em um cgroup com um único comando.

Os cgroups são um componente-chave dos contêineres porque geralmente há vários processos em execução em um contêiner que precisam ser controlados juntos.

----

Agora que entendemos o significado de contêineres, temos mais duas palavras importantes para aprender:

+ Kubernetes
+ Swarm

Essas duas palavras possuem uma frase em comum:
> **orquestração de containers**

## Para que serve a orquestração de contêineres?

Basicamente, a orquestração de contêineres automatiza a implantação, o gerenciamento, a escala e a rede dos contêineres. Use a orquestração de contêineres para automatizar e gerenciar tarefas como:

>+ Provisionamento e implantação
>+ Configuração e programação
>+ Alocação de recursos
>+ Disponibilidade dos containers
>+ Escala ou remoção de containers com base no balanceamento de cargas de trabalho na infraestrutura
>+ Balanceamento de carga e roteamento de tráfego
>+ Monitoramento da integridade do container
>+ Configuração da aplicação com base no container em que ela será executada
>+ Proteção das interações entre os containers

As ferramentas de orquestração de contêineres fornecem um framework para gerenciar arquiteturas de microsserviços e contêineres em escala. Muitas delas podem ser usadas no gerenciamento do ciclo de vida dos contêineres. Algumas opções são o Kubernetes, Docker Swarm e Apache Mesos, DC/SO e Nômade.

----

Nesse tema vamos aborda sobre e praticar:

+ [x] Docker
    + [ ] Docker Swarm: Cluster do Docker
+ [ ] Kubernetes
    + [ ] Cluster
    + [ ] Plano de controle
    + [ ] Kubelet
    + [ ] Pod
+ [ ] Google Kubernetes
+ [ ] Amazon Kubernetes
+ [ ] Azure Kubernetes

----

## Fontes usadas para elaboração desse conteúdo

+ [Red Hat: Contêineres vs Vms](https://www.redhat.com/pt-br/topics/containers/containers-vs-vms){:target="_blank"}
+ [Red Hat: Orquestração de containers](https://www.redhat.com/pt-br/topics/containers/what-is-container-orchestration){:target="_blank"}
+ [etcd: O que são namespaces e cgroups](https://etcd.dev/2021/09/20/containers-o-que-sao-namespaces-e-cgroups/){:target="_blank"}
