# Solução: Exercício 4.3 - Prometheus

**Guia de Solução:**

### Passo a Passo

1. **Expor o Prometheus**: Com a *stack* kube-prometheus instalada via Helm, utilize o `port-forward` para trazer a porta do servidor localmente: `kubectl port-forward -n monitoring svc/prometheus-server 9090:80`.
2. **Acesso e Consulta**: Acesse `http://localhost:9090` no seu navegador.
3. **Escrever a Query**: Na barra de buscas do Prometheus, insira a *query* PromQL adequada. Para listar o número de pods criados por StatefulSets no namespace do Prometheus, você pode usar a métrica `kube_pod_info` filtrada: `count(kube_pod_info{created_by_kind="StatefulSet", namespace="prometheus"})`. O resultado gráfico deve apresentar o valor 3, correspondendo aos réplicas do StatefulSet em execução.
