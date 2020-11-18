# Lab 1. Deploy da sua primeira aplicação

Aprenda como implantar um app em um cluster Kubernetes hospedado no IBM Container Service.

## 0. Pré-requisitos

Certifique-se de atender aos pré-requisitos conforme descrito em [Lab 0](../Lab0/README.md)

## 1. Deploy a aplicação guestbook

Nesta parte do laboratório, implantaremos um aplicativo chamado `guestbook`
que já foi criado e enviado ao DockerHub com o nome `ibmcom/guestbook:v1`.

1. Começe executando `guestbook`:

   ```shell
   kubectl create deployment guestbook --image=ibmcom/guestbook:v1
   ```

   Esta ação demorará um pouco. Para verificar o status do aplicativo em execução, você pode usar `$ kubectl get pods`.

   Você deve ver uma saída semelhante a esta:

   ```shell
   kubectl get pods
   ```

   Eventualmente, o status deve aparecer como `Running`.

   ```shell
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
   ```

   O resultado final do comando run não é apenas o pod que contém nossos contêineres de aplicativo, mas um recurso de implantação que gerencia o ciclo de vida desses pods.

1. Uma vez que o status lido for `Running`, precisamos expor o deploy por meio de um serviço, dessa forma poderemos acesar via IP externo
   A aplicação `guestbook` aceita requisições na porta 3000.  Execute:

   ```shell
   kubectl expose deployment guestbook --type="NodePort" --port=3000
   ```

1. Para encontrar a porta usada nesse nó de trabalho, examine seu novo serviço:

   ```shell
   $ kubectl get service guestbook
   NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   guestbook   NodePort   10.10.10.253   <none>        3000:31208/TCP   1m
   ```

   Podemos ver que nosso `<nodeport>` está na porta `31208`. Podemos ver na saída o mapeamento da porta de 3000 dentro do pod exposto para a porta 31208 do cluster. Esta porta no intervalo 31000 é escolhida automaticamente e pode ser diferente para você.

1. `guestbook` agora está sendo executado em seu cluster e exposto à Internet. Precisamos descobrir onde está acessível. Os nós de trabalho em execução no serviço de contêiner obtêm endereços IP externos. Obtenha os trabalhadores para o seu cluster e observe um (qualquer um) dos IPs públicos listados na linha `<public-IP>`. Substitua `$CLUSTER_NAME` pelo nome do cluster, a menos que você tenha esta variável de ambiente definida.

   ```shell
   $ kubectl get nodes -o wide
   NAME           STATUS   ROLES           AGE   VERSION           INTERNAL-IP    EXTERNAL-IP      OS-IMAGE   KERNEL-VERSION                CONTAINER-RUNTIME
   10.185.199.3   Ready    master,worker   63d   v1.16.2+283af84   10.185.199.3   169.59.228.215   Red Hat    3.10.0-1127.13.1.el7.x86_64   cri-o://1.16.6-17.rhaos4.3.git4936f44.el7
   10.185.199.6   Ready    master,worker   63d   v1.16.2+283af84   10.185.199.6   169.47.78.51     Red Hat    3.10.0-1127.13.1.el7.x86_64   cri-o://1.16.6-17.rhaos4.3.git4936f44.el7
   ```

   Podemos ver que nosso `<public-IP>` está em `173.193.99.136`.

1. Agora que você tem o endereço e a porta, pode acessar o aplicativo no navegador da web em `<public-IP>:<nodeport>`. Nesse exemplo está em `173.193.99.136:31208`.

Parabéns, agora você implantou um aplicativo no Kubernetes!

Quando terminar, continue para o [próximo lab](../Lab2/README.md).
