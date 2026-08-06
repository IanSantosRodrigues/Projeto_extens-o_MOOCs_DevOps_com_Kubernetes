# Exercício 4.6: O projeto v2.0

**Instruções do Exercício:**

> *Nota: Este exercício aborda a escalabilidade do broadcaster de mensagens.*
> O *broadcaster* de mensagens (usando NATS) deve ser capaz de ser escalado sem enviar a mensagem várias vezes ao cliente. 
> Teste a sua infraestrutura certificando-se de que o sistema consegue rodar com 6 réplicas ativas do broadcaster sem problemas ou travamentos. 
> Lembre-se de que as mensagens só precisam ser enviadas para o serviço externo se todos os serviços estiverem funcionando corretamente. Uma mensagem perdida aleatoriamente não é um problema grave, mas o envio de uma duplicata é.
