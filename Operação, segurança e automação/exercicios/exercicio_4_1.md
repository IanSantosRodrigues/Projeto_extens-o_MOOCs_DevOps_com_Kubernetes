# Exercício 4.1: Readiness probe

**Instruções do Exercício:**

> Crie uma `ReadinessProbe` para a aplicação Ping-pong. Ela deve estar pronta quando tiver uma conexão com o banco de dados. Além disso, crie outra `ReadinessProbe` para a aplicação Log output. Ela deve estar pronta quando puder receber dados da aplicação Ping-pong. Teste se a configuração funciona aplicando todos os manifestos do cluster, *exceto* o statefulset do banco de dados. O status READY dos pods deve refletir a falta do banco de dados antes dele ficar disponível.
