# Pipeline de Implantação e Publicação

## O que você aprenderá nesta página
- Construir e publicar imagens de contêiner
- Automatizar deploy com pipeline
- Reduzir comandos manuais na entrega da aplicação

Publicar uma aplicação manualmente funciona nas primeiras vezes, mas rapidamente vira fonte de erro. Um pipeline de implantação organiza o caminho entre alteração de código, construção da imagem, publicação no registry e atualização dos manifestos no cluster.

## Fluxo básico

Um fluxo mínimo de publicação passa por quatro etapas: testar a aplicação, construir a imagem Docker, publicar a imagem em um registry e atualizar o cluster com a nova versão.

```bash
docker build -t registry.example.com/app:v1 .
docker push registry.example.com/app:v1
kubectl set image deployment/app-dep app=registry.example.com/app:v1
```

Esse exemplo é útil para entender o processo, mas em um projeto real preferimos registrar a versão no manifesto e aplicar de forma declarativa.

## Tags de imagem

Usar `latest` parece conveniente, mas dificulta rastrear qual versão está em execução. Tags baseadas em commit, versão semântica ou número de build tornam o deploy reproduzível.

```yaml
containers:
  - name: app
    image: registry.example.com/app:sha-abc123
```

## GitHub Actions como exemplo

```yaml
name: deploy
on:
  push:
    branches: [main]
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t registry.example.com/app:sha-do-commit .
      - name: Push image
        run: docker push registry.example.com/app:sha-do-commit
```

A etapa de deploy pode aplicar manifestos diretamente, mas uma evolução natural é mover isso para GitOps, como será visto na parte de operação e automação. Não coloque credenciais diretamente no arquivo do pipeline; use secrets do provedor de CI.
