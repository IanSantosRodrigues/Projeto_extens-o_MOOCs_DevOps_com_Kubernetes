# Solução: Exercício 2.5 - StatefulSet e Headless Service

**Guia de Solução:**

### Passo a Passo

1. Crie um Headless Service com `clusterIP: None`.
2. Crie um StatefulSet com `serviceName` apontando para esse Service.
3. Use duas réplicas e confirme os nomes `app-0` e `app-1`.
4. Teste resolução DNS com `nslookup app-0.nome-do-service`.
