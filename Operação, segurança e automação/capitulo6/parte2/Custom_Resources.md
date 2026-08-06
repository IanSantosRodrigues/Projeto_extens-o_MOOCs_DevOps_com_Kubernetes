# Definições de Recursos Personalizados (*Custom Resource Definitions*)

## O que você aprenderá nesta página
- Como criar as suas próprias Definições de Recursos Personalizados (CRDs - *Custom Resource Definitions*)

As Definições de Recursos Personalizados (CRDs - *Custom Resource Definitions*) são uma forma de estender o Kubernetes com nossos próprios Recursos. Já utilizamos uma grande quantidade deles; por exemplo, no Capítulo 5, utilizamos a CRD **Application** com o ArgoCD.

Eles são uma parte tão fundamental na utilização do Kubernetes que é uma boa ideia aprender a criar os nossos próprios.

Antes de começarmos, precisamos decidir o que queremos criar. Então, vamos criar um recurso que pode ser usado para configurar contagens regressivas (*countdowns*). O recurso se chamará "Countdown". Ele terá uma certa duração (*length*) e um intervalo (*delay*) entre as execuções. A execução – o que acontece cada vez que o *delay* transcorre – fica a cargo de uma imagem. Por exemplo, alguém que usar nossa CRD poderá criar um cronômetro que poste uma mensagem no Twitter toda vez que uma contagem decrescer.

Como modelo, usarei um disponibilizado na documentação oficial, portanto, vamos colar o seguinte código no arquivo *resourcedefinition.yaml*:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # o nome deve corresponder aos campos spec abaixo, e ter o formato: <plural>.<group>
  name: countdowns.stable.dwk
spec:
  # nome do grupo para uso da API REST: /apis/<group>/<version>
  group: stable.dwk
  # as opções podem ser Namespaced ou Cluster
  scope: Namespaced
  names:
    # kind (tipo) normalmente é escrito no formato CamelCased e no singular.
    kind: Countdown
    # nome plural usado na URL: /apis/<group>/<version>/<plural>
    plural: countdowns
    # nome singular usado como atalho (alias) na linha de comando (CLI)
    singular: countdown
    # nomes curtos (shortNames) permitem buscar seu recurso de forma abreviada no CLI
    shortNames:
    - cd
  # lista das versões suportadas por essa CustomResourceDefinition
  versions:
    - name: v1
      # Cada versão pode ser ligada/desligada por esta flag
      served: true
      # Apenas uma única versão deve ser marcada como storage
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                length:
                  type: integer
                delay:
                  type: integer
                image:
                  type: string
      additionalPrinterColumns:
        - name: Length
          type: integer
          description: A duração (length) do countdown
          jsonPath: .spec.length
        - name: Delay
          type: integer
          description: O intervalo de tempo (delay, em ms) entre as execuções
          jsonPath: .spec.delay
```

Agora, podemos instanciar nosso próprio Countdown no arquivo *countdown.yaml*:

```nix
apiVersion: stable.dwk/v1
kind: Countdown
metadata:
  name: doomsday
spec:
  length: 20
  delay: 1200
  image: jakousa/dwk-app10:sha-84d581d
```

E, em seguida, aplicá-lo:

```powershell
$ kubectl apply -f countdown.yaml
  countdown.stable.dwk/doomsday created

$ kubectl get cd
  NAME        LENGTH   DELAY
  doomsday    20       1200
```

Temos agora um novo recurso! Em seguida, vamos criar um controlador personalizado (*custom controller*) que vai inicializar um *pod* rodando o contêiner gerado a partir da imagem, e vai garantir que os "countdowns" sejam destruídos após o fim. Isso exigirá um pouco de codificação.

Para a implementação, escolhi utilizar o recurso **Job** (*tarefa*), que já conhecemos de capítulos anteriores. Contêineres rodados através de Jobs são projetados para rodarem apenas uma vez até o fim (*completion*). Contudo, nem os Jobs concluídos e nem os *Pods* são excluídos sozinhos, sendo mantidos apenas para que os logs de execução possam ser revisados posteriormente.

Nosso controlador precisa realizar 3 ações:
- Criar um *Job* a partir de um *Countdown*.
- Reagendar os *Jobs* até que a quantidade de execuções definidas no *Countdown* (*length*) seja completada.
- Limpar todos os *Jobs* e *Pods* logo após as execuções.

Para implementar o controlador, precisamos executar certas interações de "baixo nível" e acessar o Kubernetes diretamente usando as APIs REST.

Observando (escutando) a API do Kubernetes no endereço `/apis/stable.dwk/v1/countdowns?watch=true`, receberemos um status `ADDED` para cada novo objeto Countdown criado no cluster. Feito isso, criar a tarefa consiste em compilar a mensagem contendo as propriedades JSON e realizar um `POST` com a carga de dados (payload) válida para `/apis/batch/v1/namespaces/<namespace>/jobs`.

No que se refere às tarefas (*Jobs*), ficaremos monitorando `/apis/batch/v1/jobs?watch=true` e esperaremos pelo evento `MODIFIED` no qual o estado de sucesso seja igual a *true*, e atualizaremos os rótulos (*labels*) dos *jobs* para armazenar seus respectivos *status*. Para apagar um *Job* e seu respectivo *Pod*, basta mandar requisições "delete" para `/api/v1/namespaces/<namespace>/pods/<pod_name>` e para `/apis/batch/v1/namespaces/<namespace>/jobs/<job_name>`.

Finalmente, é só realizar uma chamada "delete" para `/apis/stable.dwk/v1/namespaces/<namespace>/countdowns/<countdown_name>` a fim de deletar o *countdown*.

O *core* (coração) do script reside em um loop principal chamado `maintainStatus`, que resume as chamadas de maneira programática:

```oxygene
  Começa a observar os recursos Countdown (fluxo contínuo de eventos):
    Para cada evento de countdown:
      Extrai a identidade/contexto mínimos
      Se o evento for ADDED:
        Se um *job* correspondente já existir, não faça nada
        Senão, crie o primeiro/próximo job para aquela contagem
      Se o evento for DELETED:
        Execute a limpeza de toda a carga de trabalho/estado vinculada à contagem

  Começa a observar os recursos Job (fluxo contínuo de eventos):
    Para cada evento de *job*:
      Ignore os *jobs* não gerenciados por este controlador
      Ignore os *jobs* deletados/terminados
      Ignore os *jobs* que não concluíram com êxito
      Para os *jobs* gerenciados que obtiveram sucesso:
        Acione a lógica de reagendamento (crie o próximo passo na sequência)

  Mantenha ambos os *watchers* (observadores) rodando para que a reconciliação aconteça a partir dos eventos ao vivo do cluster
```

Nós não podemos simplesmente fazer a implantação, já que o código não terá permissões por padrão para consumir a API do Kubernetes. Para isso, devemos conceder o devido acesso.

## RBAC
RBAC (*Role-based access control*, ou Controle de Acesso Baseado em Funções) é uma metodologia de autorização que nos permite definir permissões de acesso para diferentes usuários, *service accounts* e/ou grupos por meio da concessão de funções (*roles*). Para nosso caso de uso, definiremos um ServiceAccount no arquivo *serviceaccount.yaml*:

```dts
apiVersion: v1
kind: ServiceAccount
metadata:
  name: countdown-controller-account
```

Em seguida, informaremos a *serviceAccountName* que a nossa aplicação utilizará em seu *deployment.yaml*:

```nix
apiVersion: apps/v1
kind: Deployment
metadata:
  name: countdown-controller-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: countdown-controller
  template:
    metadata:
      labels:
        app: countdown-controller
    spec:
      serviceAccountName: countdown-controller-account
      containers:
        - name: countdown-controller
          image: jakousa/dwk-app10-controller:sha-4256579
```

A etapa seguinte é definir as permissões necessárias e agrupá-las em "Funções" (*roles*). O Kubernetes reconhece dois tipos de *roles*: o `ClusterRole` e o `Role`. Enquanto a permissão atribuída a um `Role` é restrita aos limites de um único namespace (*namespace-specific*), o `ClusterRole` abrange todo o cluster (neste caso, é isso que precisamos, pois nosso controlador vai consultar os "countdowns" em *todos* os *namespaces*).

As permissões das regras (*rules*) são parametrizadas a partir da união de um grupo (*apiGroup*), do alvo principal (*resource*), bem como do verbo ou dos verbos (*verbs* - Get, Post, Watch, List etc) vinculados a eles.

A API no caso é `/apis/batch/v1/jobs?watch=true`, o *apiGroup* será o "batch", o *resource* as "jobs", e os verbos a lista daquilo que queremos autorizar. Quando o *apiGroup* estiver vazio (""), isso sinaliza ao Kubernetes que os recursos em questão vêm diretamente das APIs centrais. As definições ficarão parecidas com estas em um arquivo *clusterrole.yaml*:

```nix
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: countdown-controller-role
rules:
- apiGroups: [""]
  # no nível HTTP, o nome do recurso para acessar Pods é "pods"
  resources: ["pods"]
  verbs: ["get", "list", "delete"]
- apiGroups: ["batch"]
  # no nível HTTP, o nome do recurso para acessar Jobs é "jobs"
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: ["stable.dwk"]
  resources: ["countdowns"]
  verbs: ["get", "list", "watch", "create", "delete"]
```

E por último, mas não menos importante, conectaremos a "Role" recém declarada na nossa "ServiceAccount". Assim como nos tipos de Funções (*roles*), a *RoleBinding* limita-se a um único namespace, e uma *ClusterRoleBinding* é transversal e se ramificará por toda a estrutura do Kubernetes. Note que você pode conceder permissões de um recurso "ClusterRole" ligadas por uma "RoleBinding", sendo a diferença que os privilégios da referida permissão estarão *somente* atrelados àquele *namespace* isolado da "RoleBinding".

No nosso projeto, utilizaremos o `ClusterRoleBinding` para garantir acesso a todos os namespaces disponíveis. Segue o código (*clusterrolebinding.yaml*):

```nix
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: countdown-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: countdown-controller-role
subjects:
- kind: ServiceAccount
  name: countdown-controller-account
  namespace: default
```

Após fazer a implantação de todos os arquivos, poderemos testar nossos *logs* para um recurso "countdown":

```xquery
$ kubectl logs countdown-controller-dep-7ff598ffbf-q2rp5
  > app10@1.0.0 start /usr/src/app
  > node index.js

  Scheduling new job number 20 for countdown doomsday to namespace default
  Scheduling new job number 19 for countdown doomsday to namespace default
  ...
  Countdown ended. Removing countdown.
  Doing cleanup
```

**Nota: Escolha de Linguagem para os CRDs:** Os exemplos disponíveis para as CRDs são na maior parte desenvolvidos em JavaScript ou [Go](https://go.dev/). A principal linguagem de programação adotada mundialmente para escrever códigos *controllers* Kubernetes robustos costuma ser a linguagem Go (com *frameworks* de auxílio como Kubebuilder / Operator SDK).

### Exercício 5.1: Faça você mesmo um CRD e Controller
Precisamos de um recurso que extraia automaticamente links HTML como conteúdo (vamos dar o nome de *DummySite*) e gere uma cópia de sites baseado nas urls fornecidas.

- Crie a propriedade `website_url` dentro de sua nova *CustomResourceDefinition*.
- Crie o "controller" que obterá e analisará todos os *DummySites* criados a partir das APIS REST do Kubernetes.
- Faça o controlador gerar e preencher as requisições *Job* de que seu código precisará para concretizar os alvos.
