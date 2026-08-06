# Solução: Exercício 4.6 - O projeto, passo 23

**Guia de Solução:**

### Passo a Passo

1. **Publicar Mensagens (Backend)**: Integre a biblioteca do NATS no backend principal. Logo após inserir/atualizar um Todo no banco de dados com sucesso, utilize `nc.publish('todo_events', JSON.stringify(todoData))`.
2. **Criar o Serviço Broadcaster**: Crie um novo microsserviço (ex: Node, Python, Go) do zero. Este serviço conectará ao NATS e fará a inscrição (*subscribe*) no tópico `todo_events`.
3. **Encaminhar para Serviço Externo**: Quando o *broadcaster* receber uma mensagem da fila, ele deverá fazer uma requisição POST HTTP (ex: usando `axios` ou `curl`) para a URL de um *webhook* do Discord/Telegram (ou um *webhook* de testes genérico) com os dados formatados.
4. **Implantação no K8s**: Crie um `Deployment` para o *broadcaster* garantindo que ele tenha a URL do NATS e a URL do webhook injetadas via variáveis de ambiente.
