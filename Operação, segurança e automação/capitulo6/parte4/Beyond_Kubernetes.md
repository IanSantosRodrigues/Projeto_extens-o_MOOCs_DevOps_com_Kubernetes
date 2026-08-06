# Muito além do Kubernetes (*Beyond Kubernetes*)

## O que você aprenderá nesta página
- Opções de plataformas baseadas em Kubernetes
- Configurar uma plataforma *serverless* (Knative) e fazer o *deploy* de uma carga de trabalho simples

Como o Kubernetes é, na verdade, uma plataforma de base, vamos explorar alguns sistemas e blocos de construção famosos que rodam sobre ele.

> "Kubernetes é uma plataforma para construir plataformas. É um lugar melhor para se começar, mas não é o objetivo final." — Kelsey Hightower (@kelseyhightower), 27 de Novembro de 2017.

O [OpenShift](https://www.openshift.com/) é um Kubernetes com foco corporativo ("*enterprise*") - veja a [Visão geral do Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview). Afirmar que você "não usa Kubernetes porque usa OpenShift" seria o equivalente a dizer ["Eu não tenho um motor, eu tenho um carro!"](https://www.openshift.com/blog/enterprise-kubernetes-with-openshift-part-one). Outras opções de Kubernetes prontos para uso em produção incluem o [Rancher](https://rancher.com/) - que você deve ter visto anteriormente graças ao [k3d (k3s)](https://github.com/rancher/k3d) - e o [Anthos GKE](https://cloud.google.com/anthos/gke) do Google, que também deve soar familiar. Essas são todas opções a serem consideradas quando você precisa tomar a decisão crucial entre qual distribuição de Kubernetes adotar e se deseja usar um serviço gerenciado.

### Exercício 5.5: Comparação de Plataformas
Escolha um provedor de serviços (como o Rancher, por exemplo) e compare-o com outro (como o OpenShift). Decida de maneira arbitrária qual provedor de serviço é "melhor" e apresente argumentos em defesa de sua escolha contra a do outro provedor. 

*Para a submissão, uma lista em tópicos (bullet points) é o suficiente.*

## Serverless

O uso de arquiteturas [*Serverless*](https://en.wikipedia.org/wiki/Serverless_computing) ("sem servidor") ganhou muita popularidade, e é fácil entender o motivo. Quer seja o Google Cloud Run, Knative, OpenFaaS, OpenWhisk, Fission ou Kubeless, todos eles rodam em cima do Kubernetes - ou ao menos são capazes disso. Quanto mais antiga for a plataforma *serverless*, maior a probabilidade de ela não estar rodando no Kubernetes. Por isso, uma afirmação como "O Kubernetes está competindo com o serverless" não faz muito sentido.

Como este não é um curso focado em Serverless, não nos aprofundaremos nisso, embora o conceito seja bastante interessante. É por isso que, a seguir, vamos configurar uma plataforma serverless em cima do nosso k3d. Para isso, escolheremos o [Knative](https://knative.dev/), pois ele é a solução na qual o [Google Cloud Run](https://cloud.google.com/blog/products/serverless/knative-based-cloud-run-services-are-ga) é baseado, apresentando-se como uma opção bastante competente em comparação com outras alternativas *open-source* disponíveis. Isso também nos mantém no tema de "plataformas sobre plataformas", já que ele poderia ser usado para criar sua própria plataforma *serverless*.

O Knative possui seu próprio [contrato de execução (runtime contract)](https://github.com/knative/specs/blob/main/specs/serving/runtime-contract.md) apoiado pela comunidade. Ele descreve quais tipos de recursos uma aplicação **deve** e **deveria** ter para rodar de forma adequada na plataforma. Um requisito essencial é que a própria aplicação precisa ser *stateless* (sem manutenção de estado de conexão local) e configurável por meio de variáveis de ambiente. Esse tipo de especificação *open-source* ajuda o projeto a ganhar uma adoção mais ampla. Por exemplo, [o Google Cloud Run implementou o mesmo contrato](https://ahmet.im/blog/cloud-run-is-a-knative/).

### Exercício 5.6: Testando Serverless

Instale o componente *Knative Serving* no seu cluster k3d.
Para que o Knative funcione localmente no k3d, você precisa criar um cluster **sem** o Traefik:

```stylus
$ k3d cluster create --port 8082:30080@agent:0 -p 8081:80@loadbalancer --agents 2 --k3s-arg "--disable=traefik@server:0" --image rancher/k3s:v1.34.1-k3s1
```

*(Este comando também instala a versão 1.34 do Kubernetes, que é exigida pela versão mais recente do Knative).*

Em seguida, siga [este guia de instalação via YAML](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/). Para a camada de rede (*network layer*), você pode escolher o *Kourier*. Na seção "*Configure DNS*", selecione *Magic DNS (slip.io)*.

Você poderá se deparar com uma situação como esta no passo "*verify the installation*" (verificação de instalação):

```maxima
$ kubectl get pods -n knative-serving
NAME                                      READY   STATUS             RESTARTS      AGE
activator-67855958d-w2ws8                 0/1     Running            0             64s
autoscaler-5ff4c5d679-54l28               0/1     Running            0             64s
webhook-5446675b97-2ngh6                  0/1     CrashLoopBackOff   3 (12s ago)   64s
net-kourier-controller-58b6bf4fbc-g7dlp   0/1     CrashLoopBackOff   3 (10s ago)   55s
controller-6d8b579f9-p42dx                0/1     CrashLoopBackOff   3 (6s ago)    64s
```

Verifique os logs dos *pods* que estão falhando (CrashLoopBackOff) para descobrir como consertar o problema.

Logo após, faça os testes e exemplos sugeridos pela documentação:
- [Deploying a Knative Service](https://knative.dev/docs/getting-started/first-service/)
- [Autoscaling](https://knative.dev/docs/getting-started/first-autoscale/)
- [Traffic splitting](https://knative.dev/docs/getting-started/first-traffic-split/)

Note que você pode acessar o serviço localmente da sua máquina host usando:

```1c
curl -H "Host: hello.default.192.168.240.3.sslip.io" http://localhost:8081
```

Onde *Host* é a URL que você obteve ao rodar o comando:

```routeros
kubectl get ksvc
```

### Exercício 5.7: Faça o deploy para o serverless

Modifique o serviço Ping-pong do aplicativo "Log Output" para utilizar *serverless*.
A leitura deste guia de conversão pode ajudar: [Convert Deployment to Knative Service](https://knative.dev/docs/serving/convert-deployment-to-knative-service/).

DICAS:
- Utilize o DNS do serviço totalmente qualificado (*fully qualified*) do Kubernetes ao chamar os serviços. Em vez de usar, por exemplo, `http://pingpong`, você deve usar `http://pingpong.exercises.svc.cluster.local`. Isso evita problemas de roteamento de host na infraestrutura do Knative e assegura que os pedidos (requests) sejam devidamente resolvidos dentro do cluster.
- Você pode definir o "hostname" da função `url rewrite` se quiser que o endpoint *pingpong* fique disponível via navegador. O *HTTPRoute* pode ficar assim:

```nix
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /pingpong
      filters:
        - type: URLRewrite
          urlRewrite:
            hostname: pingpong.exercises.192.168.144.3.sslip.io
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /
```

### Exercício 5.8: O Panorama (Landscape)

Acesse e analise o documento oficial [CNCF Cloud Native Landscape (png)](https://landscape.cncf.io/images/landscape.png) (também disponível na [versão interativa](https://landscape.cncf.io/)).

Circule o logotipo de cada produto ou projeto que você já usou. Não precisa ter sido necessariamente durante este curso. O termo "usar" refere-se a projetos dos quais você tenha tido o mínimo de contato ciente de seu funcionamento. 

Após isso, use uma cor diferente para circular todas as dependências tecnológicas usadas por esses projetos principais - com exceção daqueles que você já circulou no primeiro passo. 
Por fim, faça uma lista detalhando os cenários e contextos de uso em que você precisou deles. Qualquer software rodado fora do contexto deste curso pode ser simplesmente rotulado como "uso fora do curso".

Por exemplo:
1. Eu utilizei o **HELM** para instalar o Prometheus na Parte 2 do curso.
2. Eu utilizei indiretamente o **Flannel**, visto que o k3d (através do k3s) o incorpora. Mas não faço a mínima ideia de como ele funciona por baixo dos panos.
3. Eu utilizei o **Istio** em projetos fora do curso.

Sinta-se à vontade para seguir com o nível de detalhe (uso indireto) o quão fundo você quiser, como no exemplo das dependências de k3d -> k3s -> flannel. Mas, utilize bom senso para que o resultado gráfico da imagem fique compreensível e limpo.
