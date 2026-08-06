# Introdução ao Networking

### Conteúdo
- [Introdução ao Networking](#introdução-ao-networking)
  - [Aplicação de rede simples](#aplicação-de-rede-simples)
  - [Conectando de fora do cluster](#conectando-de-fora-do-cluster)
  - [Configuração inicial](#configuração-inicial)
    - [Limitações do ambiente local](#limitações-do-ambiente-local)
  - [O que é um serviço?](#o-que-é-um-serviço)
  - [Sobre o roteamento do Kubernetes](#sobre-o-roteamento-do-kubernetes)
  - [O que é um Ingress?](#o-que-é-um-ingress)
  - [API de gateway](#api-de-gateway)

Agora de volta ao desenvolvimento! Reiniciar e acompanhar os logs foi um deleite. Em seguida, abriremos um endpoint para o aplicativo e o acessaremos via HTTP.

### Aplicação de rede simples

Vamos desenvolver nosso aplicativo para que ele tenha um servidor HTTP respondendo com dois hashes: um hash que é armazenado até que o processo seja encerrado e um hash que é específico da solicitação. O corpo da resposta pode ser algo como "Aplicativo abc123. Solicitação 94k9m2". Escolha qualquer porta para ouvir.

Um [exemplo](https://github.com/kubernetes-hy/material-example/tree/master/app2) já foi preparado. Por padrão, ele escutará na porta 3000.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app2/manifests/deployment.yaml
deployment.apps/hashresponse-dep created
```

### Conectando de fora do cluster

Podemos confirmar que o hashresponse-dep está funcionando com o comando `port-forward`. Vamos ver primeiro o nome do pod e depois fazer o port-forward:

```bash
$ kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
hashgenerator-dep-5cbbf97d5-z2ct9   1/1     Running   0          20h
hashresponse-dep-57bcc888d7-dj5vk   1/1     Running   0          19h

$ kubectl port-forward hashresponse-dep-57bcc888d7-dj5vk 3003:3000
Forwarding from 127.0.0.1:3003 -> 3000
Forwarding from [::1]:3003 -> 3000
```

Agora podemos ver a resposta de http://localhost:3003 e confirmar que está funcionando conforme o esperado.

Como vemos, `port-forward` é um comando usado no Kubernetes para encaminhar uma porta local para um pod. Isso permite que você acesse facilmente os aplicativos em execução dentro do seu cluster Kubernetes a partir da sua máquina local. O encaminhamento de porta não é destinado ao uso em produção, mas é muito útil para fins de depuração e desenvolvimento.

> **Exercício 1.5 — O projeto, etapa 3**
>
> **Instruções:** Faça o projeto responder algo a uma solicitação GET enviada para a URL do projeto. Uma página HTML simples é boa, ou você pode implantar algo mais complexo, como um aplicativo de página única.
>
> Veja aqui (abre em uma nova aba) como você pode definir variáveis de ambiente para contêineres.
>
> Use `kubectl port-forward` para confirmar se o projeto está acessível e funciona no cluster usando um navegador para acessar o projeto.

Conectar-se a um aplicativo dentro de um contêiner Docker é fácil. Se você tiver um aplicativo escutando a porta 3000, você pode simplesmente usar o `docker run` com a flag `-p 3003:3000`. Outra opção é usar a declaração de portas do Docker Compose. Infelizmente, o Kubernetes não é tão simples. Usaremos um recurso de serviço, um recurso de entrada (Ingress) ou a solução mais recente, a API Gateway.

### Configuração inicial

Como estamos executando nosso cluster dentro do Docker com o k3d, teremos que fazer alguns preparativos.

Abrir uma rota de fora do cluster para o pod não será suficiente se não tivermos meios de acessar o cluster k3d dentro dos contêineres do Docker! Vamos ver como é a configuração padrão:

```bash
$ docker ps
CONTAINER ID    IMAGE                      COMMAND                  CREATED             STATUS              PORTS                             NAMES
b60f6c246ebb    ghcr.io/k3d-io/k3d-proxy:5 "/bin/sh -c nginx-pr…"   2 hours ago         Up 2 hours          80/tcp, 0.0.0.0:58264->6443/tcp   k3d-k3s-default-serverlb
553041f96fc6    rancher/k3s:latest         "/bin/k3s agent"         2 hours ago         Up 2 hours                                            k3d-k3s-default-agent-1
aebd23c2ef99    rancher/k3s:latest         "/bin/k3s agent"         2 hours ago         Up 2 hours                                            k3d-k3s-default-agent-0
a34e49184d37    rancher/k3s:latest         "/bin/k3s server --t…"   2 hours ago         Up 2 hours                                            k3d-k3s-default-server-0
```

Vemos que o k3d preparou utilmente a porta 58264 para acessar a API do Kubernetes por meio do balanceador de carga, que faz proxy das solicitações para a porta 6443 no nó do servidor. Além disso, a porta 80 está aberta no balanceador de carga. Todas as solicitações ao balanceador de carga serão enviadas por proxy para o nó servidor do cluster.

Como vemos, nosso cluster possui atualmente um nó servidor e dois nós agentes. Para fins de teste, também desejaremos uma porta individual aberta para um único nó de agente. Vamos excluir nosso cluster antigo e criar um novo com portas específicas abertas para facilitar o acesso e os testes.

A documentação do K3D (abre em uma nova aba) nos diz como as portas são abertas: abriremos o local 8081 para 80 em `k3d-k3s-default-serverlb` e o local 8082 para 30080 em `k3d-k3s-default-agent-0`. O 30080 é escolhido quase completamente aleatoriamente, mas precisa ter um valor entre 30000-32767 para a próxima etapa:

```bash
$ k3d cluster delete
INFO[0000] Deleting cluster 'k3s-default'
...
INFO[0002] Successfully deleted cluster k3s-default!

$ k3d cluster create --port 8082:30080@agent:0 -p 8081:80@loadbalancer --agents 2
INFO[0000] Created network 'k3d-k3s-default'
...
INFO[0021] Cluster 'k3s-default' created successfully!
INFO[0021] You can now use it like this:
kubectl cluster-info

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app2/manifests/deployment.yaml
deployment.apps/hashresponse-dep created
```

Acima, o `agent:0` e o `loadbalancer` são baseados na documentação do k3d e na leitura do código-fonte [daqui](https://github.com/k3d-io/k3d/blob/11cc7979228f304025d61254eb0c0cb2745b9444/cmd/util/filter.go#L119) e [aqui](https://github.com/k3d-io/k3d/blob/main/pkg/types/types.go#L65).

Agora temos acesso através da porta 8081 ao nosso nó servidor (na verdade, todos os nós) e 8082 a um dos nossos nós agentes, porta 30080. Eles serão usados para mostrar diferentes métodos de comunicação com os servidores.

#### Limitações do ambiente local

A configuração não é perfeita, teremos uma quantidade limitada de portas disponíveis no futuro. Isso será suficiente para nossos casos de uso.

Seu sistema operacional pode suportar o uso da rede host, portanto, nenhuma porta precisa ser aberta. No entanto, não há experiência relatada com essa abordagem neste material.

### O que é um serviço?

Os recursos de implantação (Deployment) gerenciam o processo de implantação para nós, cuidando da criação dos pods. Dado que os pods do aplicativo são efêmeros e podem ser criados ou encerrados a qualquer momento, a comunicação com o aplicativo não deve depender da presença de nenhum pod específico.

Os recursos de serviço (Service) são essenciais para gerenciar a acessibilidade do aplicativo, garantindo que ele possa ser alcançado por conexões originadas tanto de fora do cluster quanto de dentro. Esses recursos lidam com o roteamento e o balanceamento de carga necessários para manter uma comunicação perfeita com o aplicativo, independentemente da natureza dinâmica da criação e do encerramento do pod.

Após a recriação do cluster, devemos criar novamente também nosso aplicativo de exemplo:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-hy/material-example/master/app2/manifests/deployment.yaml
```

O `deployment.yaml` tem a seguinte aparência:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hashresponse-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hashresponse
  template:
    metadata:
      labels:
        app: hashresponse
    spec:
      containers:
        - name: hashresponse
          image: jakousa/dwk-app2:b7fc18de2376da80ff0cfc72cf581a9f94d10e64
```

Vamos agora criar um arquivo `service.yaml` na pasta `manifests`. Precisamos do serviço para fazer as seguintes coisas:

- Declarar que queremos um Serviço
- Declarar qual porta ouvir
- Declarar o requerimento (seletor) para onde o pedido deverá ser dirigido
- Declarar a porta para onde o pedido deve ser dirigido

O arquivo `service.yaml` resultante é assim:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hashresponse-svc
spec:
  type: NodePort
  selector:
    app: hashresponse # This is the app as declared in the deployment.
  ports:
    - name: http
      nodePort: 30080 # This is the port that is available outside. Value for nodePort can be between 30000-32767
      protocol: TCP
      port: 1234 # This is a port that is available to the cluster, in this case it can be ~ anything
      targetPort: 3000 # This is the target port
```

```bash
$ kubectl apply -f manifests/service.yaml
service/hashresponse-svc created
```

Como publicamos 8082 como 30080, podemos acessá-lo agora via http://localhost:8082 .

Agora definimos um serviço com `type: NodePort`. Portas Node (abre em uma nova aba) são simplesmente portas abertas pelo Kubernetes para todos os nós, e o serviço manipulará solicitações nessa porta. NodePorts não são flexíveis e exigem que você atribua uma porta diferente para cada aplicativo. Como tal, os NodePorts não são usados em produção, mas são úteis de conhecer.

O que gostaríamos de usar em vez do NodePort seria um serviço do tipo LoadBalancer, mas isso "só" funciona com provedores de nuvem, pois configura um balanceador de carga possivelmente caro para ele. Conheceremos esse tipo no Capítulo 4.

Vamos analisar mais uma vez a configuração do aplicativo de exemplo:

**Implantação (hashresponse-dep)**
- Executa 1 réplica do contêiner `jakousa/dwk-app2`
- Pod rotulado com `app: hashresponse`
- O contêiner escuta na porta 3000

**Serviço (hashresponse-svc)**
- Tipo: NodePort — expõe o aplicativo externamente
- Seletor: `app: hashresponse` — roteia o tráfego para pods correspondentes
- Configuração da porta:
  - `targetPort: 3000` — a porta em que o contêiner escuta
  - `port: 1234` — a porta exposta pelo Serviço para acesso interno ao cluster
  - `nodePort: 30080` — a porta exposta em cada nó do cluster para acesso externo

Existem duas maneiras de acessar o aplicativo:

- **De dentro do cluster:** `hashresponse-svc:1234`
- **Da sua máquina local:** `localhost:8082` (o k3d mapeia a porta do host 8082 → NodePort 30080)

O tráfego flui da seguinte forma quando o aplicativo é acessado a partir da sua máquina local:

```
localhost:8082
  ↓
k3d load balancer
  ↓
node:30080 (NodePort)
  ↓
Pod container:3000
```

Portanto, o balanceador de carga do k3d permite o acesso ao host local durante o desenvolvimento.

### Sobre o roteamento do Kubernetes

O cluster foi criado da seguinte forma:

```bash
k3d cluster create --port 8082:30080@agent:0 -p 8081:80@loadbalancer --agents 2
```

Portanto, o `localhost:8082` é encaminhado para a porta 30080 do agente de nó `:0` dentro do cluster. E se o aplicativo estiver sendo executado em outro nó do cluster? O `@agent:0` da configuração acima determina qual nó recebe o tráfego externo vindo da sua máquina host. Depois disso, a rede do serviço Kubernetes assume o controle e roteia automaticamente para o local correto do pod.

É por isso que os serviços NodePort são tão poderosos: você não precisa saber ou se importar em qual nó o pod está. O serviço lida com todo o roteamento internamente.

> **Exercício 1.6 — O projeto, etapa 4**
>
> **Instruções:** Use um serviço NodePort para habilitar o acesso ao projeto.

### O que é um Ingress?

Há ainda um recurso adicional que nos ajudará a servir o aplicativo: o Ingress.

O recurso de acesso à rede de entrada, [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) , é um tipo de recurso completamente diferente dos Serviços. Se você tem o [modelo OSI](https://en.wikipedia.org/wiki/OSI_model)  memorizado, o Ingress funciona na camada 7, enquanto os serviços funcionam na camada 4. Você pode vê-los usados juntos: primeiro o LoadBalancer mencionado anteriormente e depois o Ingress para lidar com o roteamento. No nosso caso, como não temos um balanceador de carga disponível, podemos usar o Ingress como primeira parada. Se você está familiarizado com proxies reversos como o Nginx, o Ingress deve parecer familiar.

Os Ingress são implementados por vários "controladores" diferentes. Isso significa que os Ingress não funcionam automaticamente em um cluster, mas lhe dão a liberdade de escolher qual controlador Ingress funciona melhor para você. O K3s já tem o [Traefik](https://traefik.io/traefik) instalado. Outras opções incluem Istio e o Nginx Ingress Controller, entre outras, mais [aqui](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

Mudar para o Ingress exigirá que criemos um recurso do tipo Ingress. O Ingress roteará o tráfego de entrada para um Serviço, mas o antigo Serviço NodePort não fará isso, então o removeremos:

```bash
$ kubectl delete -f manifests/service.yaml
service "hashresponse-svc" deleted
```

Um recurso de Serviço do tipo [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#type-clusterip) fornece ao Serviço um IP interno que estará acessível dentro do cluster.

A seguinte definição de `service.yaml` permitirá o tráfego TCP da porta 2345 para a porta 3000:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hashresponse-svc
spec:
  type: ClusterIP
  selector:
    app: hashresponse
  ports:
    - port: 2345
      protocol: TCP
      targetPort: 3000
```

O segundo recurso que precisamos é o novo Ingress, que será definido no arquivo `ingress.yaml`. Ele precisa:

- Declarar que deve ser um Ingress
- Encaminhar todo o tráfego para o nosso serviço

A configuração é assim:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dwk-material-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hashresponse-svc
            port:
              number: 2345
```

Então podemos aplicar tudo e visualizar o resultado:

```bash
$ kubectl apply -f manifests
ingress.networking.k8s.io/dwk-material-ingress created
service/hashresponse-svc configured

$ kubectl get svc,ing
NAME                       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   10.43.0.1    <none>        443/TCP    5m22s
service/hashresponse-svc   ClusterIP   10.43.0.61   <none>        2345/TCP   45s

NAME                                             CLASS    HOSTS   ADDRESS                            PORTS   AGE
ingress.networking.k8s.io/dwk-material-ingress   <none>   *       172.21.0.3,172.21.0.4,172.21.0.5   80      16s
```

Podemos ver que o Ingress está escutando na porta 80. Como já abrimos essa porta, podemos acessar o aplicativo em http://localhost:8081.

O fluxo de tráfego ao acessar o aplicativo agora é o seguinte:

```
http://localhost:8081/
  ↓
[Ingress Controller] (port 8081 mapped from host)
  ↓
[Ingress Rules] (path: / → hashresponse-svc:2345)
  ↓
[Service] (port 2345 → targetPort 3000)
  ↓
[Pod] (container listening on port 3000)
  ↓
[Application]
```

> **Exercício 1.7 — Acesso externo com Ingress**
>
> **Instruções:** O aplicativo "Log output" atualmente gera um registro de data e hora e uma sequência aleatória (que ele cria na inicialização) para os logs.
>
> Adicione um endpoint para solicitar o status atual (timestamp e a string aleatória) e um Ingress para que você possa acessá-lo com um navegador.
>
> Você pode simplesmente armazenar a string aleatória na memória.

> **Exercício 1.8 — O projeto, etapa 5**
>
> **Instruções:** Mude para usar Ingress em vez de NodePort para acessar o projeto. Você pode excluir o Ingress do aplicativo "Log output" para que eles não interfiram neste exercício. Analisaremos mais sobre caminhos e roteamento no próximo exercício e, nesse ponto, você pode configurar o projeto para ser executado com o aplicativo "Log output" lado a lado.

> **Exercício 1.9 — Mais serviços**
>
> **Instruções:** Desenvolva um segundo aplicativo que simplesmente responda com "pong 0" a uma solicitação GET e aumente um contador (o 0) para que você possa ver quantas solicitações foram enviadas. O contador deve estar na memória para que possa ser reiniciado em algum momento. Crie uma nova implantação para ele e faça com que ele compartilhe a entrada com o aplicativo "Log output". Rotas de solicitações direcionadas a `/pingpong` para ele.
>
> Em exercícios futuros, esta segunda aplicação será chamada de "aplicação de pingue-pongue". Ela será usada com o aplicativo "Log output".
>
> O aplicativo de pingue-pongue precisará escutar solicitações em `/pingpong`, então talvez você precise fazer alterações em seu código. Isso pode ser evitado configurando o Ingress para reescrever o caminho, mas isso fica como um exercício opcional. Você pode conferir a [documentação de Ingress do Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource).

### API de gateway

No Capítulo 4, encontraremos outra maneira de acessar nosso cluster: a API Gateway, que é um conjunto evolutivo de recursos e configurações projetados para gerenciar o tráfego de rede dentro dos clusters do Kubernetes. O objetivo é fornecer uma maneira mais expressiva e extensível de definir como o tráfego deve ser roteado e controlado, em comparação com as soluções descritas nesta página.