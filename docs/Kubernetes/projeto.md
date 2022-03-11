# Projeto

Que tal um projeto integrando toda a explicação até aqui?

Vamos usar o [docker samples](https://github.com/dockersamples/k8s-wordsmith-demo){:target="_blank"}.

Crie um .yaml:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: words-db
spec:
  ports:
    - port: 5432
      targetPort: 5432
      name: db
  selector:
    app: words-db
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  labels:
    app: words-db
spec:
  selector:
    matchLabels:
      app: words-db
  template:
    metadata:
      labels:
        app: words-db
    spec:
      containers:
      - name: db
        image: dockersamples/k8s-wordsmith-db
        ports:
        - containerPort: 5432
          name: db
---
apiVersion: v1
kind: Service
metadata:
  name: words
  labels:
    app: words-api
spec:
  ports:
    - port: 8080
      targetPort: 8080
      name: api
  selector:
    app: words-api
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: words
  labels:
    app: words-api
spec:
  replicas: 5
  selector:
    matchLabels:
      app: words-api
  template:
    metadata:
      labels:
        app: words-api
    spec:
      containers:
      - name: words
        image: dockersamples/k8s-wordsmith-api
        ports:
        - containerPort: 8080
          name: api
---
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    app: words-web
spec:
  ports:
    - port: 8081
      targetPort: 80
      name: web
  selector:
    app: words-web
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: words-web
spec:
  selector:
    matchLabels:
      app: words-web
  template:
    metadata:
      labels:
        app: words-web
    spec:
      containers:
      - name: web
        image: dockersamples/k8s-wordsmith-web
        ports:
        - containerPort: 80
          name: words-web
```

Em seguida:

=== "Execute"

    ```bash
    kubectl apply -f kube-deployment.yml
    ```

    Para verificar os serviços ativos:

    ```bash
    kubectl get svc
    ```

=== "Saida"

    ```bash
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    db           ClusterIP      None            <none>        5432/TCP         17s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          3d23h
    svc-pod      ClusterIP      10.105.54.122   <none>        9000/TCP         24h
    svc-pod2     NodePort       10.98.89.245    <none>        5000:30000/TCP   61m
    web          LoadBalancer   10.97.254.156   <pending>     8081:31555/TCP   17s
    words        ClusterIP      None            <none>        8080/TCP         17s
    ```

No meu caso ele redirecionou o Load Balance para porta **31555**, ficando assim **192.168.49.2:31555**. Se precisar do **INTERNAL-IP**, basta executar **kubectl get nodes -o wide**.
