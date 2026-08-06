# Solução: Exercício 2.3 - ConfigMaps e Secrets

**Guia de Solução:**

### Passo a Passo

1. Crie um ConfigMap com dados como `APP_ENV` e `API_URL`.
2. Crie um Secret com a senha codificada em Base64 ou via `kubectl create secret generic`.
3. No Deployment, use `envFrom.configMapRef` e `secretKeyRef`.
4. Valide com logs ou endpoint da aplicação, sem imprimir a senha em texto claro.
