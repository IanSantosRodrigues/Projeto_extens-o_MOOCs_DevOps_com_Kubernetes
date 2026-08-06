# Solução: Exercício 4.2 - O projeto, passo 21

**Guia de Solução:**

### Passo a Passo

1. **Criar Rota Healthz no Backend**: Crie uma variável global `isHealthy = true`. Adicione uma rota `GET /healthz` que retorne código `200 OK` se `isHealthy` for verdadeiro, ou `500 Internal Server Error` se for falso.
2. **Simulação de Falha**: Crie um botão no Frontend (ex: 'Quebrar App') e faça-o acionar uma rota no Backend que altera `isHealthy` para `false`.
3. **Configurar as Probes**: No manifesto de Deployment da aplicação, adicione `readinessProbe` e `livenessProbe` apontando para o *path* `/healthz` na porta do seu servidor web.
4. **Verificação**: Implante as mudanças. Ao clicar no botão 'Quebrar App', a aplicação começará a retornar erro 500 no `/healthz`. O Kubernetes (kubelet) notará a falha através da `livenessProbe` e forçará a reinicialização (restart) do *pod*, restaurando a saúde do serviço.
