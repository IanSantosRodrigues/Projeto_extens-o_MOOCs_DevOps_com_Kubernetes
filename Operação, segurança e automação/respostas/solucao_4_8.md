# Solução: Exercício 4.8 - O Projeto, etapa 24

**Guia de Solução:**

### Passo a Passo

1. **Preparar os Manifestos do Projeto**: Coloque todos os recursos (Deployment, Service, Ingress, StatefulSet, Secrets/ConfigMaps) de O Projeto em uma estrutura Kustomize no GitHub.
2. **Registrar a Aplicação no ArgoCD**: Faça o mesmo passo do exercício anterior, agora definindo a `Application` do projeto.
3. **Configuração de Destino e Fonte**: Defina o `repoURL` do seu repositório de projeto, o `targetRevision` como `HEAD` (ou `main`) e o `path` para a pasta correta.
4. **Verificação de Sync**: Crie o recurso via `kubectl apply -n argocd -f project-application.yaml`. Toda vez que você fizer um `git push` no repositório com novas tags de imagens Docker, o Argo detectará a diferença e fará o *deploy* dos novos componentes silenciosamente.
