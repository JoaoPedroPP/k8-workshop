# Lab 0. 

## Configure seu ambiente Kubernetes

Para os laboratórios deste repositório, você precisará de um cluster kubernetes. Uma opção para criar um cluster é usar o Kubernetes como um serviço do IBM Cloud Kubernetes Service, conforme descrito abaixo.

### Use o IBM Cloud Kubernetes Service

Você precisará de uma conta paga do IBM Cloud ou de uma conta do IBM Cloud que seja uma conta de teste (diferente da conta Lite). Se você tiver uma dessas contas, use o [Guia de introdução](https://cloud.ibm.com/docs/containers?topic=containers-getting-started) para criar seu cluster.

### Use um ambiente de teste hospedado

Existem alguns serviços pela Internet para uso temporário. Como esses serviços são gratuitos, às vezes podem ocorrer períodos de disponibilidade/qualidade limitada. Por outro lado, podem ser uma forma rápida de começar!

* [Kubernetes playground on Katacoda](https://www.katacoda.com/courses/kubernetes/playground) Este ambiente começa com um nó mestre e um nó de trabalho pré-configurados. Você pode executar as etapas do Labs 1 em diante a partir do nó mestre.

* [Play with Kubernetes](https://labs.play-with-k8s.com/) Depois de fazer login com seu github ou id do hub do docker, clique em **Start**, **Add New Instance** e siga as etapas mostradas no terminal para ativar o cluster e adicionar trabalhadores.

### Configure na sua máquina

Se desejar configurar kubernetes para execução em sua estação de trabalho local para não produção e uso de aprendizado, há várias opções.

* [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) Esta solução requer a instalação de um provedor de VM compatível (KVM, VirtualBox, HyperKit, Hyper-V - dependendo da plataforma)

* [Kubernetes in Docker (kind)](https://kind.sigs.k8s.io/) Executa um cluster do Kubernetes em Docker containers

* [Docker Desktop (Mac)](https://docs.docker.com/docker-for-mac/kubernetes/) [Docker Desktop (Windows)](https://docs.docker.com/docker-for-windows/kubernetes/) O Docker Desktop inclui um ambiente kubernetes

* [Microk8s](https://microk8s.io/docs/) Kubernetes instaláveis empacotados como uma imagem `snap` do Ubuntu.

# Instale a IBM Cloud CLI

1. Como pré-requisito para o plug-in IBM Cloud Kubernetes Service, instale a [IBM Cloud command-line interface](https://clis.ng.bluemix.net/ui/home.html). Depois de instalado, você pode acessar o IBM Cloud a partir de sua linha de comando com o prefixo `ibmcloud`.
2. Faça o log in no IBM Cloud CLI: `ibmcloud login`.
3. Insira suas credenciais da IBM Cloud quando solicitado.

   **Obs:** Se você tiver um ID federado, use`ibmcloud login --sso` para logar na IBM Cloud CLI. Insira seu nome de usuário e use a URL fornecida na saída da CLI para recuperar sua senha única. Você sabe que tem um ID federado quando o login falha sem o`--sso` efunciona com `--sso`.

# Instale o plug-in da IBM Cloud Kubernetes Service
1. Para criar clusters Kubernetes e gerenciar nós de trabalho, instale o plug-in IBM Cloud Kubernetes Service:
   ```ibmcloud plugin install kubernetes-service```

   **Obs:** O prefixo para executar comandos usando o plug-in IBM Cloud Kubernetes Service é `ibmcloud ks`.

2. Para verificar se o plug-in está instalado corretamente, execute o seguinte comando:
```ibmcloud plugin list```

   O plug-in IBM Cloud Kubernetes Service é exibido nos resultados como `container-service/kubernetes-service`.

# Download o Kubernetes CLI

Para visualizar uma versão local do painel do Kubernetes e implantar aplicativos em seus clusters, você precisará instalar a CLI do Kubernetes que corresponde ao seu sistema operacional:

* [OS X](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/darwin/amd64/kubectl)
* [Linux](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/linux/amd64/kubectl)
* [Windows](https://storage.googleapis.com/kubernetes-release/release/v1.10.8/bin/windows/amd64/kubectl.exe)

**Para usuário de Windows:** Instale o Kubernetes CLI no mesmo diretório que o IBM Cloud CLI. Esta configuração evita algumas mudanças no caminho do arquivo quando você executa comandos posteriormente.

**Para MacOS e Linux:**

1. Mova o executável para o diretório `/usr/local/bin` usando o comando `mv /<path_to_file>/kubectl /usr/local/bin/kubectl` .

2. Certifique-se de que `/usr/local/bin` está listado em sua variável de sistema PATH.
```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

3. Converta o arquivo binário em executável: `chmod +x /usr/local/bin/kubectl`

# Configure o Kubectl para apontar para o serviço IBM Cloud Kubernetes
1. Liste os clusters em sua conta:

```shell
ibmcloud ks clusters
```

2. Defina uma variável de ambiente que será usada em comandos subsequentes neste laboratório.

```shell
export CLUSTER_NAME=<your_cluster_name>
```

3. Configure `kubectl` para apontar para o seu cluster
```shell
ibmcloud ks cluster config --cluster $CLUSTER_NAME
```

3. Valide se a configuração está adequada
```shell
kubectl get namespace
```

4. Você deve ver uma saída semelhante à seguinte; em caso afirmativo, você está pronto para continuar.

```shell
NAME              STATUS   AGE
default           Active   125m
ibm-cert-store    Active   121m
ibm-system        Active   124m
kube-node-lease   Active   125m
kube-public       Active   125m
kube-system       Active   125m
```

# Download the Workshop Source Code
O repositório `guestbook` tem o aplicativo que iremos implementar.
Embora não vamos construí-lo, usaremos os arquivos de configuração de implantação desse repo.
O aplicativo do livro de visitas tem duas versões v1 e v2 que usaremos para demonstrar alguns lançamentos funcionalidade mais tarde. Todos os arquivos de configuração que usamos estão no diretório guestbook/v1.

```shell
git clone https://github.com/IBM/guestbook.git
```

