# Exercício 5.3: Log app, a Edição Service Mesh

**Instruções do Exercício:**

> Implemente a aplicação Log output na malha de serviço (*service mesh*). 
> - Estenda a aplicação desenvolvendo um novo serviço chamado *greeter*, capaz de responder a uma requisição HTTP GET com uma saudação ('Hello').
> - A aplicação *log output* deve passar a consumir o serviço *greeter* via rede e exibir a saudação na tela juntamente com as saídas dos logs regulares.
> - Implante duas versões diferentes do *greeter* (v1 e v2) e configure as regras de roteamento do Istio (como o *HTTPRoute*) para dividir o tráfego: uma versão deve receber 75% das requisições e a outra os 25% restantes.
