# Instalação

Para instalar o kubernete:

=== "Windows"

    Siga o passo a passo da [documentação](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/){:target="_blank"}

=== "Linux"

    Siga o passo a passo da [documentação](https://kubernetes.io/docs/tasks/tools/){:target="_blank"}

=== "Mac"

    Siga o passo a passo da [documentação](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/){:target="_blank"}

----

Precisamos instalar uma ferramenta para ajudar a criar essa infraestrutura localmente, temos algumas opções que a [documentação sugere](https://kubernetes.io/docs/tasks/tools/){:target="_blank"} que são o [**kind**](https://kind.sigs.k8s.io/docs/user/quick-start/){:target="_blank"}, [**minikube**](https://minikube.sigs.k8s.io/docs/start/){:target="_blank"} e [**kubeadm**](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/){:target="_blank"}, vamos instalar somente o [**minikube**](https://minikube.sigs.k8s.io/docs/start/){:target="_blank"} para realizar nossos estudos.

Na documentação basta verificar o passo a passo para sua OS e fazer a instalação.

Para iniciar o **minikube**:

```bash
minikube start
```

Ele vai dizer que está faltando um [driver](https://minikube.sigs.k8s.io/docs/drivers/){:target="_blank"}, se você já tiver o docker instalado facilita o processo, mas se não basta instalar uns dos drives que ele recomenda para sua OS.

Vou usar o docker e setar ele para usar como default:

```bash
minikube config set driver docker
```

Ele solicitou para dá um start no docker daemon:

```bash
sudo systemctl start docker
```

Como isso basta, iniciar novamente o **minikube**, vamos testar o **kubectl get nodes**:

```bash
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   3m54s   v1.23.3
```

Lembrando que, futuramente vou criar cluster em serviços de nuvem, como o AWS, GCP, Azure e talvez na Oracle. Mas nesse momento, vamos focar em kubernetes e como funciona toda essa estrutura, pratica e teoricamente.
