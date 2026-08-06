# Exercício 4.2: O projeto, passo 21

**Instruções do Exercício:**

> Crie as sondas de prontidão e vivacidade (`ReadinessProbes` e `LivenessProbes`), bem como o *endpoint* necessário (como `/healthz`) para O Projeto, de forma a garantir que a aplicação esteja funcionando corretamente e conectada ao banco de dados.
> Adicione um botão à sua interface que pode ser usado para 'quebrar' a aplicação (simular uma falha). Pressionar o botão deve fazer com que a operação normal pare.
> Certifique-se de que, uma vez quebrado, o Kubernetes identifique a falha e um novo pod inicie, ficando saudável novamente.
