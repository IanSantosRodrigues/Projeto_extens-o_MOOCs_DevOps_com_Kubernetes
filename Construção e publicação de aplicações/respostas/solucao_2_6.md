# Solução: Exercício 2.6 - Monitoramento básico

**Guia de Solução:**

### Passo a Passo

1. Verifique métricas rápidas com `kubectl top pods` e `kubectl top nodes` quando houver Metrics Server.
2. Para Prometheus/Grafana, prefira namespace `monitoring` e instalação via Helm.
3. Observe reinícios com `kubectl get pods` e detalhes com `kubectl describe pod`.
4. Registre quais métricas indicam saúde da aplicação: CPU, memória, READY, restarts e latência.
