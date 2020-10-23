
# Pod

No Kubernetes, um grupo de um ou mais container é chamado de pod. Os container em um pod são implantados juntos e iniciados, interrompidos e replicados como um grupo. A definição de pod mais simples descreve a implantação de um único contêiner. Por exemplo, um pod de servidor da web nginx pode ser definido como:
```
apiVersion: v1
kind: Pod
metadata:
  name: mynginx
  namespace: default
  labels:
    run: nginx
spec:
  containers:
  - name: mynginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

# Labels

No Kubernetes, as labels são um sistema para organizar objetos em grupos. As labels são pares de valores-chave anexados a cada objeto. Os seletores de rótulo podem ser passados junto com uma solicitação ao apiserver para recuperar uma lista de objetos que correspondem a esse seletor de rótulo.

Para adicionar um rótulo a um pod, adicione uma seção de labels nos metadados na definição do pod:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
...
```
Para adicionar um label a um pod em execução
```
  kubectl label pod mynginx type=webserver
  pod "mynginx" labeled
```
Para listar os pods com base nas labels
```
  kubectl get pods -l type=webserver
  NAME      READY     STATUS    RESTARTS   AGE
  mynginx   1/1       Running   0          21m

```


# Deployments

Um Deployment fornece declarações declarativas para pods e réplicas. Você só precisa descrever o estado desejado em um objeto Deployment e ele mudará o estado real para o estado desejado. O objeto Deployment define os seguintes detalhes:

Os elementos de uma definição de controlador de replicação
A estratégia de transição entre deployments
Para criar uma deployment para um servidor da web nginx, edite o arquivo nginx-deploy.yaml como
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  generation: 1
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30

```
and create the deployment
```
kubectl create -f nginx-deploy.yaml
deployment "nginx" created
```
The deployment creates the following objects
```
kubectl get all -l run=nginx

NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/nginx   3         3         3            3           4m

NAME                 DESIRED   CURRENT   READY     AGE
rs/nginx-664452237   3         3         3         4m

NAME                       READY     STATUS    RESTARTS   AGE
po/nginx-664452237-h8dh0   1/1       Running   0          4m
po/nginx-664452237-ncsh1   1/1       Running   0          4m
po/nginx-664452237-vts63   1/1       Running   0          4m
```

# services

Services

Os pods do Kubernetes, como contêineres, são efêmeros. Replication Controllers criam e destroem pods dinamicamente, por exemplo, ao aumentar ou diminuir ou ao fazer atualizações contínuas. Embora cada pod obtenha seu próprio endereço IP, mesmo esses endereços IP não podem ser considerados estáveis ao longo do tempo. Isso leva a um problema: se algum conjunto de pods fornece funcionalidade para outros pods dentro do cluster do Kubernetes, como esses pods descobrem e controlam quais outros?

Um Kubernetes Service é uma abstração que define um conjunto lógico de pods e uma política para acessá-los. O conjunto de pods direcionados por um serviço geralmente é determinado por um seletor de rótulos. O Kubernetes oferece uma API Endpoints simples que é atualizada sempre que o conjunto de pods em um serviço muda.

To create a service for our nginx webserver, edit the nginx-service.yaml file
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  selector:
    run: nginx
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 80
  type: ClusterIP
```
Criar um serviço

`kubectl create -f nginx-service.yaml`
service "nginx" created
```
kubectl get service -l run=nginx
NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx     10.254.60.24   <none>        8000/TCP    38s

```
Descrever um serviço
```
kubectl describe service nginx
Name:                   nginx
Namespace:              default
Labels:                 run=nginx
Selector:               run=nginx
Type:                   ClusterIP
IP:                     10.254.60.24
Port:                   <unset> 8000/TCP
Endpoints:              172.30.21.3:80,172.30.4.4:80,172.30.53.4:80
Session Affinity:       None
No events.
```
O serviço acima está associado aos nossos pods nginx anteriores. Preste atenção ao campo do seletor de serviço run = nginx. Ele informa ao Kubernetes que todos os pods com o rótulo run=nginx estão associados a este serviço e devem ter o tráfego distribuído entre eles. Em outras palavras, o serviço fornece uma camada de abstração e é o ponto de entrada para alcançar todos os pods associados.
