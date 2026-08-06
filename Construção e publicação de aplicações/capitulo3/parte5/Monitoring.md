# Monitoramento de Aplicações no Kubernetes

## O que você aprenderá nesta página
- Entender por que monitorar aplicações publicadas
- Coletar métricas com Prometheus e visualizar com Grafana
- Relacionar métricas com escalabilidade e diagnóstico

Publicar uma aplicação não termina quando o `kubectl apply` funciona. Depois que ela está em execução, precisamos responder perguntas simples e importantes: a aplicação está saudável? O consumo de CPU subiu? Os Pods estão reiniciando? O banco está respondendo? O tráfego aumentou?

Monitoramento é a prática de coletar, armazenar e visualizar sinais sobre o sistema. No Kubernetes, esses sinais vêm dos próprios recursos do cluster, dos contêineres e da aplicação.

## Métricas do cluster

O Metrics Server fornece métricas básicas para comandos como:

```bash
kubectl top pods
kubectl top nodes
```

Ele é útil para uma visão rápida e também serve de base para recursos como HPA em muitos ambientes. Porém, para análises históricas e painéis completos, normalmente usamos Prometheus e Grafana.

## Prometheus e Grafana

Prometheus coleta métricas periodicamente a partir de endpoints HTTP. Grafana permite montar painéis para visualizar essas métricas. Em ambientes Kubernetes, uma instalação comum é feita com Helm usando o chart `kube-prometheus-stack`.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

## Métricas úteis para publicação

Durante a construção e publicação de aplicações, algumas métricas são especialmente importantes: reinícios de contêineres, quantidade de Pods prontos, consumo de CPU e memória, latência, taxa de erro e disponibilidade de banco de dados. Essas métricas ajudam a decidir se uma versão pode continuar em produção ou se precisa ser revertida.

## Boas práticas

Configure endpoints de saúde, use readiness e liveness probes, acompanhe reinícios e mantenha dashboards simples. Um painel bom não é o que mostra tudo; é o que ajuda a perceber rapidamente quando algo importante saiu do normal.
