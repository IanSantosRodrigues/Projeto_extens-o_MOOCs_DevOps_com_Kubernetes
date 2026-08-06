# Solução: Exercício 4.1 - Readiness Probe

**Guia de Solução:**

### Passo a Passo

1. **Adicionar Readiness Probe ao Ping-pong**: Edite o manifesto `Deployment` da aplicação Ping-pong. Adicione o campo `readinessProbe` para verificar a conexão com o banco de dados (ex: executando um script `pg_isready` ou acessando um *endpoint* que só responde quando o banco está acessível).
2. **Adicionar Readiness Probe ao Log Output**: No `Deployment` do Log Output, configure um `readinessProbe` do tipo `httpGet` para tentar acessar o serviço do Ping-pong na porta correta.
3. **Teste Prático**: Faça o apply de todos os manifestos, exceto o do banco de dados (StatefulSet/Service). Verifique o status com `kubectl get pods`. Os *pods* do Ping-pong e do Log Output devem iniciar, mas o campo `READY` ficará como `0/1` porque a sonda de prontidão falhará. Assim que você fizer o apply do banco de dados, eles passarão para `1/1`.
