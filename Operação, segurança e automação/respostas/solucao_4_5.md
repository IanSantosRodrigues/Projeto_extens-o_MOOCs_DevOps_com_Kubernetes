# Solução: Exercício 4.5 - O projeto, passo 22

**Guia de Solução:**

### Passo a Passo

1. **Modificação no Banco de Dados**: Adicione uma coluna/campo booleano `done` (concluído) na sua tabela ou esquema de tarefas (Todos), com valor padrão `false`.
2. **Atualizar o Backend**: Crie o endpoint `PUT /todos/:id` no backend. O controlador dessa rota deve buscar a tarefa correspondente no banco de dados e atualizar o valor de `done` para `true`.
3. **Atualizar o Frontend**: Na interface de usuário, adicione um botão 'Concluído' ou um *checkbox* ao lado de cada tarefa listada. Ao ser clicado, o botão envia a requisição HTTP PUT para a nova rota da API.
4. **Implantação**: Atualize as imagens Docker, altere as tags no *Deployment* e aplique no Kubernetes. Valide a funcionalidade criando uma tarefa e depois marcando-a como concluída.
