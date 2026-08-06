# Solução: Exercício 2.2 - Namespaces

**Guia de Solução:**

### Passo a Passo

1. Crie os namespaces com `kubectl create namespace dev` e `kubectl create namespace prod`.
2. Aplique os manifestos usando `-n dev` e `-n prod` ou declare `metadata.namespace`.
3. Compare `kubectl get svc -n dev` com `kubectl get svc -n prod`.
4. Teste DNS entre namespaces usando `nome-do-service.nome-do-namespace`.
