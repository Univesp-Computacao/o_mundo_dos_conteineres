# O que é um POD?

Os pods são as menores unidades de computação implantáveis ​​que você pode criar e gerenciar no Kubernetes.

Um Pod é um grupo de um ou mais contêineres, com armazenamento compartilhado e recursos de rede e uma especificação de como executar os contêineres. O conteúdo de um pod é sempre co-localizado e co-agendado e executado em um contexto compartilhado. Um Pod modela um "host lógico" específico do aplicativo: ele contém um ou mais contêineres de aplicativos que são relativamente fortemente acoplados. Em contextos fora da nuvem, os aplicativos executados na mesma máquina física ou virtual são análogos aos aplicativos em nuvem executados no mesmo host lógico.

Além dos contêineres de aplicativos, um Pod pode conter contêineres init que são executados durante a inicialização do Pod. Você também pode injetar contêineres efêmeros para depuração se seu cluster oferecer isso.

O contexto compartilhado de um Pod é um conjunto de namespaces do Linux, cgroups e potencialmente outras facetas de isolamento - as mesmas coisas que isolam um contêiner do Docker. Dentro do contexto de um Pod, os aplicativos individuais podem ter mais subisolamentos aplicados.

Em termos de conceitos do Docker, um Pod é semelhante a um grupo de contêineres do Docker com namespaces compartilhados e volumes de sistema de arquivos compartilhados, ou seja, não vamos manipular diretamente os containers e sim os PODs, que vai se encarregar de criar os containers.

Os PODs possuem endereços IPs, isso significa que independente de quantos containers estejam no POD, existe somente um endereço IP que é o do POD, e as portas que são liberadas pelos containers, são sempre acessadas a partir do mesmo endereço, mesmo que seja de containers diferentes. Lembrando que os essa estrutura de endereçamento de IP não permite que dois containers possuam a mesma porta. Os PODs podem se comunicar entre sí, assim como os containers que agoram compartilha o mesmo IP do POD que estão alocado, possuem o mesmo IP. possibilitando a comunicação via localhost.

![Exemplo](https://i.postimg.cc/NMTRtmgM/image.png){ loading=lazy }

----

## Primeiro POD

Para inicialziar um POD, precisamos ir na documentação da API e lá vamos encontrar esse comando [kubectl run](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run){:target="_blank"}, com isso já podemos ir no terminal para iniciar nosso primeiro POD.

```bash
kubectl run testando --image=nginx
```

Para verificar os PODs que estão ativos [kubectl get pods](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get){:target="_blank"}, na documentação temos as flags que podemos aplicar, por exemplo o **--watch** que visualizar os pods, da criação até efetivar a ativação.

Se precisamos de mais informações do POD basta usar [kubectl describe pod](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe){:target="_blank"} e passa o nome do POD, que no nosso caso é **testando**:

```bash
kubectl describe pod testando
```

Com isso, temos informação do IP do POD, como ele está e os eventos que sucederam após sua criação. Que tal deletamos esse POD com [kubectl delete pod](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete){:target="_blank"}.

Essa é a criação simples de um POD, porém não é a mais recomendada, a documentação do kubernetes enfatiza o uso de arquivos com as informações sobre a imagem para criação do POD, podemos usar tanto o JSON quanto o yaml, nesse caso, por recomendação da kubernetes devemos usar conforme as [melhores praticas](https://kubernetes.io/pt-br/docs/concepts/configuration/overview/){:target="_blank"} o yaml.

Abra uma pasta e crie um arquivo **o_primeiro_pod.yaml**, pode nomeaĺo como quiser, mas tem que ter o final .yaml.

```YAML
apiVersion: v1
kind: Pod
metadata:
    name: o-primeiro-pod
spec:
    containers:
        - name: container-nginx
          image: nginx
```

Com o arquivo já com as informações abra o terminal na pasta que você inseriu o arquivo, e execute [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply){:target="_blank"} com a flag **-f** para passa o nome do arquivo que queremos executar.

```bash
kubectl apply -f ./o_primeiro_pod.yaml
```

E criamos nosso primeiro POD de modo declarativo.

----

## Serviços

Os PODs possuem uma comunicação interna, e que é gerenciando de forma autonoma por um recurso chamado **service** ou **svc**, [documentação do service](https://kubernetes.io/docs/concepts/services-networking/service/){:target="_blank"}, é importante entender o funcionamento do **svc** pois ele vai ser usado dentro do nosso cluster para definir comunicação entre os PODs, entre suas tarefas estão:

+ Abstração para expor aplicações em um POD ou mais.
+ Proveem IPs fixos para comunicação
+ Proveem DNS em um POD ou mais
+ Fazem o balanceamento de carga

E **service** possui quatro tipos internos que são:

+ ClusterIP: Expõe o serviço em um IP interno no cluster. Esse tipo torna o serviço acessível apenas de dentro do cluster.
+ NodePort: Expõe o serviço na mesma porta de cada nó selecionado no cluster usando NAT. Torna um serviço acessível de fora do cluster usando <NodeIP>:<NodePort>. Superconjunto de ClusterIP.
+ Load Balance: Cria um balanceador de carga externo na nuvem atual (se compatível) e atribui um IP externo fixo ao serviço. Superconjunto de NodePort.
+ ExternalName: Mapeia o serviço para o conteúdo do campo externalName (por exemplo, foo.bar.example.com), retornando um registro CNAME com seu valor. Nenhum proxy de qualquer tipo é configurado. Este tipo requer v1.7 ou superior de kube-dns, ou CoreDNS versão 0.0.8 ou superior.

----

### ClusterIP

Para criar um ClusterIP:

=== "POD - 1"

    Crie uma pasta

    Dentro da pasta crie um arquivo, vou nomealo de pod-1.yaml

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
        labels:
            app: pod-label
    spec:
        containers:
            - name: pod-1
              image: nginx
              ports:
                - containerPort: 80
    ```

=== "Service"

    Dentro da pasta crie um arquivo, vou nomealo de svc-pod.yaml

    ```YAML
    apiVersion: v1
    kind: Service
    metadata:
        name: svc-pod
    spec:
        type: ClusterIP
        selector:
            app: pod-label
        ports:
            - port: 4050
              targetPort: 80
    ```

    Repare que no **spec** especificamos um **ClusterIP** e não um container, e em **selector** estamos dizendo para o Serviço procura um rótulo/labels nos PODs. Que é o link de conexão, é essa informação que faz o trabalho de criar a conexão. 

    Em **ports** estou dizendo para entrar a informação/requisão na porta 4050 e dizendo para sair **targetPort** na porta 80.

    E em kind, estamos passando a informação de **Service** e não **Pod**

----

Agora, vamos subir os serviços e depois os PODs, sempre nessa ordem.

=== "1"

    ```bash
    kubectl apply -f svc-pod.yaml
    ```

=== "2"

    ```bash
    kubectl apply -f pod-1.yaml
    ```

----

Para verificar os IPs internos usamos o [kubectl get pods](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get){:target="_blank"} e passamos a flag **-o** e **wide**:

=== "Execute"

    ```bash
    kubectl get pods -o wide
    ```

=== "Saida"

    ```bash
    NAME             READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
    o-primeiro-pod   1/1     Running   0          5m37m   172.17.0.3   minikube   <none>           <none>
    pod-1            1/1     Running   0          2m26s   172.17.0.4   minikube   <none>           <none>
    ```

----

Para verificar o cluster IP precisamo usar o [kubectl get svc](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get){:target="_blank"}.

```bash
kubectl get svc
```

    ```bash
    NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    2d23h
    svc-pod      ClusterIP   10.105.54.122   <none>        4050/TCP   9m4s
    ```

Com isso a comunicação interna do Cluster está com um fluxo, se houvesse mais PODs, eles conseguiriam se comunicar com o POD1 atraves do ClusterIP que definimos, porém o POD1.

----

### NodePort

Criando um NodePort.

=== "POD - 2"

    Dentro da pasta crie um arquivo, vou nomealo de pod-1.yaml

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-2
        labels:
            app: pod-label2
    spec:
        containers:
            - name: pod-2
              image: sposigor/app-flask-teste:1
              ports:
                - containerPort: 5000
    ```

=== "Service"

    Dentro da pasta crie um arquivo, vou nomealo de svc-pod.yaml

    ```YAML
    apiVersion: v1
    kind: Service
    metadata:
        name: svc-pod2
    spec:
        type: NodePort
        ports:
            - port: 5000
              nodePort: 30000
        selector:
            app: pod-label2
    ```

## NÃO FINALIZADO
