# Solução: Exercício 4.6 - O projeto v2.0

**Guia de Solução:**

### Passo a Passo

1. **Identificar o Problema**: Ao escalar o serviço *broadcaster* (que assina os tópicos do NATS), várias réplicas receberão a mesma mensagem, gerando envios duplicados.
2. **Uso de Queue Groups no NATS**: Na biblioteca ou cliente NATS utilizado no seu código, modifique a inscrição (*subscription*) para pertencer a um `Queue Group`.
3. **Exemplo de Código**: Se for Node.js, em vez de `nc.subscribe('todo_topic', (msg) => {...})`, altere para `nc.subscribe('todo_topic', { queue: 'broadcaster-workers' }, (msg) => {...})`.
4. **Implantação e Teste**: Atualize o código, gere a imagem, mude as réplicas do Deployment do Broadcaster para 6 (`replicas: 6`). Teste criar uma tarefa. Graças ao Queue Group, o NATS balanceará o tráfego, garantindo que apenas 1 das 6 réplicas processe a mensagem.
