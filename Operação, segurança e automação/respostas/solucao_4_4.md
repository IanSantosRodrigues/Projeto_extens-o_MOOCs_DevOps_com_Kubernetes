# Solução: Exercício 4.4 - Seu canário

**Guia de Solução:**

### Passo a Passo

1. **Definição do AnalysisTemplate**: Crie um arquivo YAML com a estrutura de `AnalysisTemplate` fornecida pelo Argo Rollouts. Nomeie-o, por exemplo, como `cpu-usage-rate`.
2. **Configuração da Métrica**: Defina o provedor como `prometheus` e aponte para o endereço DNS do serviço Prometheus no cluster (ex: `http://prometheus-server.monitoring.svc.cluster.local:80`).
3. **Query PromQL**: No campo `query`, escreva a consulta de uso da CPU dos contêineres: `sum(rate(container_cpu_usage_seconds_total{namespace="default", pod=~"pingpong-.*"}[5m]))`.
4. **Condição de Sucesso**: Defina a condição de sucesso (`successCondition`) como `result < 0.5` (ou um valor razoável para a sua aplicação).
5. **Vinculação ao Rollout**: No recurso de `Rollout` da aplicação Ping-pong, insira um passo de `analysis` logo após o primeiro `setWeight`, referenciando o `cpu-usage-rate`. Se o uso estourar o limite, o *rollback* será automático.
