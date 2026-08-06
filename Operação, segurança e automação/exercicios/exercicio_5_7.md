# Exercício 5.7: Faça o deploy para o serverless

**Instruções do Exercício:**

> Modifique o serviço Ping-pong do seu aplicativo 'Log Output' para utilizar o modelo *serverless* implantando-o como um Knative Service.
> - Utilize a nomenclatura de DNS do serviço totalmente qualificado (*fully qualified name*, ex: `http://pingpong.exercises.svc.cluster.local`) para garantir que os demais componentes locais consigam se comunicar com ele na infraestrutura do Knative.
> - Defina o `hostname` nas regras do `URLRewrite` caso queira testar a exposição do endpoint *pingpong* e torná-lo acessível no seu navegador.
