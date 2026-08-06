# Exercício 4.4: Seu canário

**Instruções do Exercício:**

> Crie um `AnalysisTemplate` (utilizado pelo Argo Rollouts) para a aplicação Ping-pong que acompanhará e medirá o uso de CPU de todos os contêineres do seu *namespace*.
> A lógica de análise deve verificar se a taxa de uso da CPU (a soma) subir acima de um valor predefinido num intervalo de 5 minutos. Se o valor for excedido, o *rollout* deve ser considerado falho e a atualização deve ser revertida (*rollback*) automaticamente.
