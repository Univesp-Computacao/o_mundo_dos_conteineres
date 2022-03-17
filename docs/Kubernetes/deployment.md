# Deployment

>Você descreve um estado desejado em um Deployment e a Deployment controla e altera o estado real para o estado desejado a uma taxa controlada. Você pode definir Deployment para criar novos ReplicaSets ou remover Deployment existentes e adotar todos os seus recursos com novas Deployment.

Bem, para simplificar iremos criar um .yaml e passar no kind o Deployment, que vai criar um versionamento para a aplicação, porém ele engloba o ReplicaSets, passando o a quantidade de replicas.

![deployment](https://i.postimg.cc/4xrtynDg/image.png){ loading=lazy }


Para simplicar ainda mais é praticamente um ReplicaSet com versionamento.

Crie o arquivo .yaml e vamos executar:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Execute em seguida:

```bash
kubectl apply -f deployment.yaml
```

Para verificar:

```bash
kubectl get deployment
```

----

## Versionamento

Com o versionamento podemos, criar versões e depois podemos retorna se houve necessidade

=== "Execute"

    ```bash
    kubectl rollout history deployment
    ```

    Se houver mais deployment basta passa o nome no final para especificar.

=== "Saida"

    ```bash
    REVISION  CHANGE-CAUSE
    1         <none>
    ```

    Aqui no nosso historico está em none na versão 1


Vou alterar a versão do **nginx:1.14.2** para **nginx:1** com o set.

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1
```

Em seguida vamos repetir o processo para verificar a revisão.

=== "Execute"

    ```bash
    kubectl rollout history deployment
    ```

    Se houver mais deployment basta passa o nome no final para especificar.

=== "Saida"

    ```bash
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```

    Aqui no nosso historico agora com a versão 2

Que tal mudamos o nome das alterações que fizemos? Usando o **annotate**

```bash
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="alteração da imagem para a versão 1"
```

E a saida do **kubectl rollout history deployment** agora está com a alteração:

```bash
REVISION  CHANGE-CAUSE
1         <none>
2         alteração da imagem para a versão 1
```

E para retorna para alguma versão especifica:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

## Escalando a quantidade de replicas

Para escalar é bem simples:

```bash
kubectl scale deployment/nginx-deployment --replicas=10
```

Usando o **scale** e com a flag **--replicas** definimos a quantidade de replicas.