# ReplicaSet

>A finalidade de um ReplicaSet é manter um conjunto estável de pods de réplica em execução a qualquer momento. Como tal, é frequentemente usado para garantir a disponibilidade de um número especificado de Pods idênticos.

![replicaset](https://i.postimg.cc/Vv6N76BH/image.png){ loading=lazy }
----

## Como funciona os ReplicaSet

Um ReplicaSet é definido com campos, incluindo um seletor que especifica como identificar os pods que ele pode adquirir, um número de réplicas indicando quantos pods ele deve manter e um modelo de pod especificando os dados dos novos pods que ele deve criar para atender ao número de critérios de réplicas. Um ReplicaSet cumpre sua finalidade criando e excluindo Pods conforme necessário para atingir o número desejado. Quando um ReplicaSet precisa criar pods, ele usa seu modelo de pod.

Um ReplicaSet é vinculado a seus pods por meio do campo metadata.ownerReferences dos pods, que especifica de qual recurso o objeto atual pertence. Todos os Pods adquiridos por um ReplicaSet têm as informações de identificação de seu próprio ReplicaSet no campo ownerReferences. É por meio desse link que o ReplicaSet conhece o estado dos Pods que está mantendo e planeja de acordo.

Um ReplicaSet identifica novos pods a serem adquiridos usando seu seletor. Se houver um pod que não tenha OwnerReference ou OwnerReference não for um Controlador e corresponder ao seletor de um ReplicaSet, ele será imediatamente adquirido pelo referido ReplicaSet.

----

## Quando usar um ReplicaSet?

Um ReplicaSet garante que um número especificado de réplicas de pod esteja em execução a qualquer momento. No entanto, uma implantação é um conceito de nível superior que gerencia ReplicaSets e fornece atualizações declarativas para Pods com muitos outros recursos úteis. Portanto, recomendamos o uso de implantações em vez de usar diretamente ReplicaSets, a menos que você exija uma orquestração de atualização personalizada ou não exija nenhuma atualização.

Na verdade, isso significa que talvez você nunca precise manipular objetos ReplicaSet: use um Deployment e defina seu aplicativo na seção spec.

----

## Configurando o replicaset

Vamos usar o exemplo disponivel na documentação:

```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

Antes de continuar, precisamos entender melhor cada parte dessa configuração feita.

=== "Parte 1"

    ```YAML
    kind: ReplicaSet
    metadata:
      name: frontend
      labels:
        app: guestbook
        tier: frontend
    ```

    Estamos passando para o **kind** o **ReplicaSet** para o kubernetes entender ser um serviço do tipo **ReplicaSet**

    Em **metadata** mais especificamente no **labels** foi inserido o **tier** que é um nome para um conjunto, então imagine que tem varios PODs e ao invés de criar um **matchLabels** para cada app, criamos um **tier** e inserimos essa informação em todos os PODs desse conjunto, assim precisamos criar somente um **ReplicaSet** para todos esses PODs.

=== "Parte 2"

    ```YAML
    spec:
      # modify replicas according to your case
      replicas: 3
      selector:
        matchLabels:
          tier: frontend
      template:
        metadata:
          labels:
            tier: frontend
        spec:
          containers:
          - name: php-redis
            image: gcr.io/google_samples/gb-frontend:v3
    ```

    **replicas** é a quantidade de PODs ideal para aplicação, mas também é a quantidade de PODs que sobem. Teremos um número ideal de PODs disponíveis e a quantidade de PODs que estão efetivamente disponíveis e nesse momento o **ReplicaSet** faz sua mágica, sendo a substituição automática dos PODs que efetivamente caem.

    **selector** estamos falando para ele procurar todos os PODs, do **tier** **frontend** para realizar o processo de monitorar, replicar e substituir.

    **template** é o padrão que vai conter as informações do POD.

----

Com isso, basta criar o arquivo.yaml e executar.

```bash
kubectl apply -f replicas.yaml
```

Ele vai executar o comando e vamos verificar:

```bash
kubectl get rs
```

Saida:

```bash
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       3s
```

Temos o nome do **ReplicaSet** que criamos, a quantidade de **Desired** que é um número desejado informado em **replicas** o **Current** que é a informação sobre os PODs que estão normais e o **Ready** que está informando que estão todos ativos.

### Testando o ReplicaSet

Com a mesma aplicação, observaremos na prática como ele funciona, primeiro abra dois terminas e deixe um do lado do outro.

No primeiro vamos digitar o **kubectl get rs** e passar o parâmetro **--watch** para conseguimos verificar em tempo real.

```bash
kubectl get rs --watch
```

No outro terminal, faremos primeiro um **kubectl get pod** e em seguida **kubectl delete pod** e passe um dos PODs disponíveis.

=== "get pod"

    ```bash
    kubectl get pod
    ```

    ```bash
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-hvvnp   1/1     Running   0          6m53s
    frontend-t4l87   1/1     Running   0          6m53s
    frontend-xqfsq   1/1     Running   0          6m53s
    ```

    Vou usar o **frontend-hvvnp** para o delete

=== "delete"

    ```bash
    kubectl delete pod frontend-hvvnp
    ```


Após concluir o procedimento, perceba que o outro terminal teve uma alteração:

```bash
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       8m18s
frontend   3         2         2       8m50s
frontend   3         3         2       8m50s
frontend   3         3         3       8m51s
```

Aqui estamos vendo que o valor de **Current** e **Ready** tiveram mudança e em seguida retornaram ao valor original.

Com isso nosso teste está concluído.

### Balanceamento de Carga

O **ReplicaSet** também se integra no balanceamento de carga, visto que o serviço pode ser distribuído igualmente pelas réplicas disponíveis.

Para verificar mais informações consultar a [Documentação](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/){:target="_blank"}
