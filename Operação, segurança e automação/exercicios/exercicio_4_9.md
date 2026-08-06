# Exercício 4.9: O projeto, etapa 25

**Instruções do Exercício:**

> Melhore as definições de *deployment* do seu Projeto da seguinte forma com Kustomize e ArgoCD:
> - Crie dois ambientes separados (*production* e *staging*) e os instancie em *namespaces* diferentes no cluster.
> - Cada commit aprovado na ramificação `main` deve acionar automaticamente o *deploy* no ambiente de *staging*.
> - Cada commit que possuir uma **tag** de versão (*release*) deve acionar o *deploy* no ambiente de *production*.
> - No ambiente de *staging*, o *broadcaster* apenas deve registrar as mensagens em formato de logs, e não deverá enviá-las de verdade para os serviços de chat externos.
> - No ambiente de *staging*, não configure ou execute backups do banco de dados.
