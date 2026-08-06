# GitOps

## O que você aprenderá nesta página
- Comparar GitOps a outros métodos de implantação (*deployment*)
- Comparar a implantação tradicional baseada em *push* (*push deployment*) com a implantação baseada em *pull* (*pull deployment*)
- Implementar GitOps no seu cluster Kubernetes com o ArgoCD

Um pipeline de implantação simples padrão inclui as seguintes etapas:
1. O desenvolvedor envia (*pushes*) o código modificado para um repositório, como o GitHub.
2. Isso aciona um serviço de CI/CD, como o GitHub Actions, para iniciar a execução.
3. O serviço de CI/CD executa os testes, compila uma imagem, envia a imagem para um registro e implanta a nova imagem, por exemplo, no Kubernetes.

Isso é chamado de **implantação baseada em push** (*push deployment*). O nome é descritivo, pois tudo é "empurrado" (*pushed*) para a frente pelo passo anterior. Existem alguns desafios com a abordagem de *push*. Por exemplo, se tivermos um cluster Kubernetes que não está disponível para conexões externas, ou seja, o cluster na sua máquina local ou qualquer cluster ao qual não queremos dar acesso a terceiros. Nesses casos, não é possível fazer com que o CI/CD envie a atualização para o cluster.

Na **configuração de pull** (*pull configuration*), o processo é revertido. O cluster, rodando em qualquer lugar, **busca** (*pull*) a nova imagem e a implantação ocorre automaticamente. A nova imagem ainda será testada e compilada pelo CI/CD. Nós simplesmente aliviamos o CI/CD do fardo da implantação e transferimos a responsabilidade de "buscar" os dados (*pulling*) para um sistema separado.

### Watchtower
Se você concluiu o curso de DevOps com Docker, deve ter ouvido falar sobre o [Watchtower](https://github.com/containrrr/watchtower), que nos permitiu transformar os "pushes" finais do pipeline de implantação em "pulls" quando rodamos serviços com docker-compose.

**GitOps** trata-se exatamente desta inversão, promovendo boas práticas para o lado operacional. Isto é alcançado mantendo o estado do cluster em um repositório Git, gerenciando, portanto, não só implantações de aplicações, mas também todas as mudanças de infraestrutura no cluster. Isso exigirá configuração adicional e uma mudança de mentalidade, superando a tradição de configurações convencionais em servidores. Mas, quando chegarmos a esse ponto, o GitOps será o último prego no caixão do gerenciamento imperativo de clusters.

O [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) é atualmente a principal ferramenta para GitOps em Kubernetes, e é a nossa escolha para este módulo. No fim das contas, nosso fluxo de trabalho deve ficar assim:
1. O desenvolvedor executa *git push* com código ou configurações modificadas.
2. O serviço de CI/CD (GitHub Actions, no nosso caso) inicia a execução.
3. O serviço de CI/CD compila e faz o *push* da nova imagem **e** confirma a alteração na ramificação principal (*main*, no nosso caso).
4. O ArgoCD obterá o estado descrito na *branch* de release (ramo de liberação) e configurará esse estado no nosso cluster.

Vamos iniciar instalando o ArgoCD seguindo a [documentação do Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/):

```awk
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Agora o ArgoCD está sendo executado no nosso cluster. Ainda precisamos abrir o acesso a ele. Existem várias opções; usaremos um LoadBalancer, com o seguinte comando:

```scilab
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Após uma breve espera, o cluster nos fornece um endereço de IP externo:

```armasm
$ kubectl get svc -n argocd
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
...
argocd-server      LoadBalancer   10.7.5.82     34.88.152.2   80:32029/TCP,443:30574/TCP   17min
```

A senha inicial para a conta *admin* é gerada automaticamente e armazenada em texto claro no campo `password` na *secret* chamada `argocd-initial-admin-secret`, presente na configuração de *namespace* do ArgoCD. Portanto, decodificamos a string em base64 com este comando:

```pgsql
kubectl get -n argocd secrets argocd-initial-admin-secret -o yaml
```

Com o ArgoCD rodando e logado, vamos tentar implementar a aplicação simples que está disponível no repositório [dwk-app1](https://github.com/mluukkai/dwk-app1) usando o ArgoCD. Faça um *fork* desse repositório se quiser seguir o exemplo!

Começamos clicando em **Create application** e preenchendo o formulário. Por padrão, a configuração do *sync policy* fica em **manual**, e também configuramos o `path` apontando diretamente para `.` (ponto) no repositório Git, já que a configuração (*kustomization.yaml*) está no diretório raiz.

A aplicação é criada, mas constará como ausente e fora de sincronia (*Missing and out of sync*), visto que o modo foi configurado como manual. Após sincronizá-la e ir para a página da aplicação no ArgoCD, poderemos ver um erro (representado por um símbolo de "coração partido" em um pod). Podemos visualizar mais detalhes usando `kubectl get po` ou clicando sobre o pod no painel do ArgoCD.

O problema é que especificamos uma imagem que não existe. Vamos arrumar isso no GitHub alterando nosso *kustomization.yaml* da seguinte forma:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- manifests/deployment.yaml
- manifests/service.yaml
images:
- name: PROJECT/IMAGE
  newName: mluukkai/dwk1
```

Quando sincronizarmos novamente as mudanças no ArgoCD, um novo pod funcional iniciará e nosso app passará a rodar!

Se mudarmos a política de sincronização de manual para **automática** na interface do Argo, qualquer mudança no GitHub será aplicada automaticamente no cluster. Teste aumentar o número de réplicas no *deployment.yaml* e faça o commit; o ArgoCD irá sincronizar as mudanças por conta própria (frequência de checagem a cada 180 segundos) e escalar a aplicação.

O repositório Git (o *source*) torna-se a única fonte da verdade, contendo o *deployment.yaml*, *service.yaml* e o *kustomization.yaml*. Qualquer edição à imagem da aplicação passará primeiramente pelas ferramentas do Kustomize e pelo pipeline no GitHub Action:

```yaml
name: Build and publish application

on:
  push:

jobs:
  build-publish:
    name: Build, Push, Release
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Login to Docker Hub
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Crie a imagem com a tag SHA do GitHub para obter uma tag exclusiva
      - name: Build and publish backend
        run: |-
          docker build --tag "mluukkai/dwk1:$GITHUB_SHA" .
          docker push "mluukkai/dwk1:$GITHUB_SHA"

      - name: Set up Kustomize
        uses: imranismail/setup-kustomize@v3

      - name: Use right image
        run: kustomize edit set image PROJECT/IMAGE=mluukkai/dwk1:$GITHUB_SHA

      - name: commit kustomization.yaml to GitHub
        uses: EndBug/add-and-commit@v10
        with:
          add: 'kustomization.yaml'
          message: New version released ${{ github.sha }}
```

Para que isso funcione, também é necessário permitir permissões de gravação de repositório (*write permissions*) na configuração de fluxo de trabalho (*workflow*) do GitHub. E agora temos uma integração GitOps 100% automatizada! 

Os benefícios da implantação com GitOps incluem:
- **Melhor Segurança:** Ninguém precisa de acesso ao cluster, nem os serviços de CI/CD. Não há necessidade de compartilhar chaves com terceiros; todos apenas confirmam (*commit*) o código no Git.
- **Maior Transparência:** Todo o escopo é definido no GitHub. Se uma pessoa nova entrar no time, ela só precisará verificar o repositório, não sendo necessário repassar conhecimentos ocultos.
- **Maior Rastreabilidade:** Todas as mudanças no cluster têm controle de versão. É fácil saber quem fez as mudanças e como ficou o estado.
- **Redução de Risco:** Caso ocorra algum erro e as coisas quebrem, usar um comando `git revert` é o suficiente para voltar à normalidade do cluster.
- **Portabilidade:** Ao mudar de provedor de Nuvem, basta montar um novo cluster e apontá-lo para o mesmo repositório - pronto, seu cluster está implementado novamente.

### Exercício 4.7: Passos de bebê para GitOps
Mova o aplicativo *Log output* para usar o GitOps para que, ao fazer o *commit* no repositório, o aplicativo seja atualizado automaticamente.

### Exercício 4.8: O Projeto, etapa 24
Mova O Projeto (*The Project*) para usar o GitOps de maneira que, após um *commit*, a aplicação seja atualizada automaticamente. Neste exercício, basta que a *branch main* (ramificação principal) faça o *deploy* no cluster.

---

## Múltiplos ambientes (More environments)

Se um aplicativo possuir muitos ambientes, como produção (*production*), teste (*staging*) e desenvolvimento, defini-los de forma direta envolve muito copia-e-cola (*copy-paste*). Com o uso do Kustomize, a recomendação é definir uma "base" (*base*) comum para todos e em seguida utilizar as "camadas" (*overlays*) relativas a cada ambiente, onde apenas as partes diferentes estarão configuradas.

Estrutura de diretório:
```stylus
.
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays
    ├── prod
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── staging
        └── kustomization.yaml
```

As partes fixas permanecem em `base`, já em `overlays/prod/deployment.yaml` alteramos por exemplo, as réplicas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep
spec:
  replicas: 3
```

E no `overlays/prod/kustomization.yaml` usamos as tags `patches` para aplicar essas modificações específicas de produção na base:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ./../../base
patches:
  - path: deployment.yaml

namePrefix: prod-

images:
  - name: PROJECT/IMAGE
    newName: nginx:1.31-alpine
```

## Definindo o todo com YAML (Defining the whole with yaml)

Até agora usamos a UI do ArgoCD para definir *applications*. Porém, isso também pode ser feito de modo declarativo e definindo uma configuração como um recurso do Kubernetes.
Para a versão de produção, a configuração `application.yaml` ficará assim:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/mluukkai/gitopstest
    path: overlays/prod
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Depois de declarar tudo num único YAML, apenas integre via `kubectl apply -n argocd -f application.yaml`.

### Exercício 4.9: O projeto, etapa 25
Melhore as definições do seu Projeto da seguinte forma:
- Crie dois ambientes separados (*production* e *staging*) em *namespaces* diferentes.
- Cada commit na ramificação `main` deve fazer o *deploy* no *staging*.
- Cada commit **com tag** deve fazer o *deploy* no ambiente *production*.
- Em staging, o *broadcaster* apenas registra mensagens (logs); não deverá envia-las para serviços externos.
- Em staging, o banco de dados não recebe *backups*.

### Exercício 4.10: O projeto, o grand finale
Finalize a infraestrutura do seu projeto, fazendo com que sejam utilizados repositórios totalmente distintos: um para os códigos-fonte da aplicação e outro unicamente para as configurações de *deployment*.
