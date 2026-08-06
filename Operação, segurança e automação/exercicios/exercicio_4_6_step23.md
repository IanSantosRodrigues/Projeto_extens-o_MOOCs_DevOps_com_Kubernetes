# Exercício 4.6: O projeto, passo 23

**Instruções do Exercício:**

> *Nota: Este exercício aborda a criação do Broadcaster NATS.*
> Crie um novo serviço separado para enviar mensagens de status das tarefas (todos) para algum serviço de chat externo. Vamos chamar o novo serviço de 'broadcaster'.
> Requisitos: 
> 1. O backend deve enviar uma mensagem para a fila do NATS ao salvar ou atualizar uma tarefa;
> 2. O serviço *broadcaster* deve assinar (subscribe) as mensagens do NATS; 
> 3. O *broadcaster* deve repassar a mensagem para um serviço externo em um formato suportado (Você pode integrar com Discord, Telegram, Slack, ou usar um endpoint genérico que registre os logs do payload).
