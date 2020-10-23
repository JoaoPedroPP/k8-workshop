# Lab 3: Escale e Atualize um app nativo, criando uma aplicação multi-tier.

Neste laboratório, você aprenderá como implantar o mesmo aplicativo guestbook que implantamos nos laboratórios anteriores, no entanto, em vez de usar as funções auxiliares de linha de comando `kubectl`, implantaremos o aplicativo usando arquivos de configuração. O mecanismo do arquivo de configuração permite que você tenha um controle mais refinado sobre todos os recursos que estão sendo criados no cluster do Kubernetes.

Antes de trabalharmos com o aplicativo, precisamos clonar um repositório github:

```shell
git clone https://github.com/IBM/guestbook.git
```

Este repo contém várias versões do aplicativo guestbook
bem como os arquivos de configuração que usaremos para implantar as partes do aplicativo.

Mude o diretório executando o comando
```shell
cd guestbook/v1
```
Você encontrará todos os arquivos de configuração para este exercício neste diretório.

## 1. Escale aplicativos nativamente

O Kubernetes pode implantar um pod individual para executar um aplicativo, mas quando você precisa escaloná-lo para lidar com um grande número de solicitações, uma `implantação` é o recurso que você deseja usar. Uma implantação gerencia uma coleção de pods semelhantes. Quando você pede um número específico de réplicas, o Kubernetes Deployment Controller tentará manter esse número de réplicas o tempo todo.

Cada objeto Kubernetes que criamos deve fornecer dois campos de objeto aninhados que regem a configuração do objeto: o objeto `spec` e o objeto `status`. O objeto `spec` define o estado desejado e o objeto `status` contém informações fornecidas pelo sistema Kubernetes sobre o estado real do recurso. Conforme descrito antes, o Kubernetes tentará reconciliar o estado desejado com o estado real do sistema.

Para Object que criamos, precisamos fornecer a `apiVersion` que você está usando para criar o objeto, `kind` do objeto que estamos criando e os `metadados` sobre o objeto, como um `name`, conjunto de `labels` e opcionalmente `namespace` ao qual este objeto deve pertencer.

Considere a seguinte configuração de implantação para o aplicativo guestbook

**guestbook-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook-v1
  labels:
    app: guestbook
    version: "1.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
        version: "1.0"
    spec:
      containers:
      - name: guestbook
        image: ibmcom/guestbook:v1
        ports:
        - name: http-server
          containerPort: 3000
```

O arquivo de configuração acima cria um objeto de implantação denominado 'guestbook' com um pod contendo um único contêiner executando a imagem ʻibmcom/guestbook:v1`. Além disso, a configuração especifica réplicas definidas como 3 e o Kubernetes tenta garantir que pelo menos três pods ativos estejam em execução o tempo todo.

- Faça o deployment do guestbook

   Para criar uma implantação usando este arquivo de configuração, usamos o seguinte comando:

   ```shell
   kubectl create -f guestbook-deployment.yaml
   ```

- Liste os pods com o label app=guestbook

  Podemos então listar os pods que ele criou listando todos os pods que têm um rótulo de "app" com um valor de "guestbook". Isso corresponde aos rótulos definidos acima no arquivo yaml na seção `spec.template.metadata.labels`.

   ```shell 
   kubectl get pods -l app=guestbook
   ```

Quando você altera o número de réplicas na configuração, o Kubernetes tenta adicionar ou remover pods do sistema para corresponder à sua solicitação. Para fazer essas modificações, use o seguinte comando:

   ```shell
   kubectl edit deployment guestbook-v1
   ```

Isso irá recuperar a configuração mais recente para o Deployment do servidor Kubernetes e, em seguida, carregá-la em um editor para você. Você notará que há muito mais campos nesta versão do que no arquivo yaml original que usamos. Isso ocorre porque ele contém todas as propriedades sobre o Deployment que o Kubernetes conhece, não apenas aquelas que escolhemos especificar ao criá-lo. Observe também que agora contém a seção `status` mencionada anteriormente.

Para sair do editor `vi`, digite `:q! `, Ou se você fez alterações que deseja ver refletidas, salve-as usando `:wq`.

Você também pode editar o arquivo de implantação que usamos para criar o Deployment para fazer alterações. Você deve usar o seguinte comando para tornar a mudança efetiva ao editar a implantação localmente.

   ```shell
   kubectl apply -f guestbook-deployment.yaml
   ```

Isso solicitará ao Kubernetes que "diferencie" nosso arquivo yaml com o estado atual do Deployment e aplique apenas essas alterações.

Agora podemos definir um objeto Service para expor a implantação a clientes externos.

**guestbook-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  ports:
  - port: 3000
    targetPort: http-server
  selector:
    app: guestbook
  type: LoadBalancer
```

A configuração acima cria um recurso Service denominado guestbook. Um Service pode ser usado para criar um caminho de rede para o tráfego de entrada para seu aplicativo em execução. Nesse caso, estamos configurando uma rota da porta 3000 no cluster para a porta "http-server" em nosso aplicativo, que é a porta 3000 de acordo com as especificações do contêiner Deployment.

- Vamos agora criar o serviço guestbook usando o mesmo tipo de comando que usamos quando criamos o Deployment:

  ```shell
  kubectl create -f guestbook-service.yaml
  ```

- Teste o aplicativo guestbook usando um navegador de sua escolha usando a url `<your-cluster-ip>:<node-port>`

  Lembre-se, para obter o `nodeport` e` public-ip` use os seguintes comandos, substituindo `$CLUSTER_NAME` pelo nome do seu cluster se a variável de ambiente ainda não estiver configurada.

  ```shell
  kubectl describe service guestbook
  ```
  and

  ```shell 
  kubectl get nodes -o wide
  ```

# 2. Conecte-se a um serviço de back-end.

Se você olhar o código-fonte do guestbook, no diretório `guestbook/v1/guestbook`, você notará que ele foi escrito para suportar uma variedade de armazenamentos de dados. Por padrão, ele manterá o registro de entradas do guestbook na memória. Isso é bom para fins de teste, mas conforme você entra em um ambiente mais "real", onde escala seu aplicativo, esse modelo não funcionará porque, com base na instância do aplicativo para a qual o usuário é roteado, eles verão resultados muito diferentes.

Para resolver isso, precisamos ter todas as instâncias de nosso aplicativo compartilhando o mesmo banco de dados - neste caso, vamos usar um banco de dados redis que implantamos em nosso cluster. Esta instância do redis será definida de maneira semelhante ao livro de visitas.

**redis-master-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
      - name: redis-master
        image: redis:3.2.9
        ports:
        - name: redis-server
          containerPort: 6379
```

Este yaml cria um banco de dados redis em um Deployment chamado 'redis-master'. Ele criará uma única instância, com réplicas definidas como 1, e as instâncias do aplicativo do livro de visitas se conectarão a ele para persistir os dados, bem como ler os dados persistentes de volta. A imagem em execução no caontainer é redis:3.2.9' e expõe a porta 6379 padrão do redis.

- Crie um redis Deployment, como fizemos para guestbook:

    ```shell
    kubectl create -f redis-master-deployment.yaml
    ```

- Verifique se o pod do servidor redis está em execução:

    ```shell
    $ kubectl get pods -lapp=redis,role=master
    NAME                 READY     STATUS    RESTARTS   AGE
    redis-master-q9zg7   1/1       Running   0          2d
    ```

- Vamos testar o pod do redis. Substitua o nome do pod `redis-master-q9zg7` pelo nome do seu pod.

    ```shell
    kubectl exec -it redis-master-q9zg7 redis-cli
    ```

    O comando kubectl exec iniciará um processo secundário no contêiner especificado. Neste caso, estamos solicitando que o comando "redis-cli" seja executado no contêiner denominado "redis-master-q9zg7". Quando esse processo termina, o comando "kubectl exec" também sai, mas os outros processos no contêiner não são afetados.

    Uma vez no container, podemos usar o comando "redis-cli" para garantir que o banco de dados redis esteja funcionando corretamente ou para configurá-lo, se necessário.

    ```shell
    redis-cli> ping
    PONG
    redis-cli> exit
    ```

Agora precisamos expor o Deployment as a Service `redis-master` para que o aplicativo guestbook possa se conectar a ele através do DNS lookup.

**redis-master-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: master
```

Isso cria um objeto de serviço chamado 'redis-master' e o configura para a porta de destino 6379 nos pods selecionados pelos seletores "app=redis" e "role=master".

- Crie o serviço para acessar o redis master:

    ```shell
    kubectl create -f redis-master-service.yaml
    ```

- Reinicie o guestbook para que ele encontre o serviço redis para usar o banco de dados:

    ```shell
    kubectl delete deploy guestbook-v1
    kubectl create -f guestbook-deployment.yaml
    ```

- Teste o aplicativo guestbook usando um navegador de sua escolha usando a url `<your-cluster-ip>:<node-port>`, ou atualizando a página se você já tiver o aplicativo aberto em outra janela.
  
Você pode ver agora que, se abrir vários navegadores e atualizar a página para acessar as diferentes cópias do @guestbook, todos eles terão um estado consistente. Todas as instâncias gravam no mesmo armazenamento persistente de apoio e todas as instâncias são lidas desse armazenamento para exibir as entradas @guestbook que foram armazenadas.

Temos nosso aplicativo simples de 3 camadas em execução, mas precisamos escalonar o aplicativo se o tráfego aumentar. Nosso principal gargalo é que temos apenas um servidor de banco de dados para processar cada solicitação proveniente de guestbook. Uma solução simples é separar as leituras e gravações de modo que elas vão para bancos de dados diferentes que são replicados adequadamente para obter consistência de dados.

![rw_to_master](../images/Master.png)

Crie uma deployment chamada 'redis-slave' que pode se comunicar com o banco de dados redis para gerenciar leituras de dados. Para escalar o banco de dados, usamos o padrão onde podemos escalar as leituras usando redis slave deployment que pode executar várias instâncias para ler. Redis slave deployments está configurado para executar duas réplicas.

![w_to_master-r_to_slave](../images/Master-Slave.png)

**redis-slave-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
      - name: redis-slave
        image: ibmcom/guestbook-redis-slave:v2
        ports:
        - name: redis-server
          containerPort: 6379
```

- Crie o pod executando redis slave deployment.
 
  ```shell
  kubectl create -f redis-slave-deployment.yaml 
  ```

 - Verifique se todas as réplicas de redis slave estão rodando

  ```shell
  $ kubectl get pods -lapp=redis,role=slave
  NAME                READY     STATUS    RESTARTS   AGE
  redis-slave-kd7vx   1/1       Running   0          2d
  redis-slave-wwcxw   1/1       Running   0          2d
  ```

- Em seguida, vá até um desses pods e examine o banco de dados para ver se tudo está certo. Substitua o nome do pod `redis-slave-kd7vx` pelo seu próprio nome de pod. Se você obtiver o retorno `(lista vazia ou conjunto)` ao imprimir as chaves, vá para o aplicativo @guestbook e adicione uma entrada!

 ```shell
$ kubectl exec -it redis-slave-kd7vx  redis-cli
127.0.0.1:6379> keys *
1) "guestbook"
127.0.0.1:6379> lrange guestbook 0 10
1) "hello world"
2) "welcome to the Kube workshop"
127.0.0.1:6379> exit
```

Deploy redis slave service para que possamos acessá-lo pelo nome DNS. Uma vez reimplantado, o aplicativo enviará operações de "leitura" para os pods `redis-slave`, enquanto as operações de" gravação "irão para os pods` redis-master`.

**redis-slave-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    role: slave
```

- Crie o serviço para acessar redis slave
    ```shell
    kubectl create -f redis-slave-service.yaml
    ```

- Reinicie o guestbook para que ele encontre o serviço slave para ler dados.
    ```shell
    kubectl delete deploy guestbook-v1
    kubectl create -f guestbook-deployment.yaml
    ```
    
- Teste o aplicativo guestbook usando um navegador de sua escolha usando o url `<your-cluster-ip>:<node-port>` ou atualizando a página se o aplicativo estiver aberto em outra janela.

É o fim do laboratório. Agora vamos limpar nosso ambiente:

```shell
kubectl delete -f guestbook-deployment.yaml
kubectl delete -f guestbook-service.yaml
kubectl delete -f redis-slave-service.yaml
kubectl delete -f redis-slave-deployment.yaml 
kubectl delete -f redis-master-service.yaml 
kubectl delete -f redis-master-deployment.yaml
```
