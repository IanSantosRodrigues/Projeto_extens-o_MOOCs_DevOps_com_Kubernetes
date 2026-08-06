# Exercício 5.1: Faça você mesmo um CRD e Controller

**Instruções do Exercício:**

> Neste exercício, precisamos criar um recurso do zero. Vamos dar o nome de *DummySite* a um recurso que extraia automaticamente o HTML de uma página da web e crie uma cópia do site.
> - Crie uma *CustomResourceDefinition* (CRD) definindo a propriedade `website_url`.
> - Desenvolva um controlador (*controller*) personalizado que obterá e analisará os eventos de todos os *DummySites* criados a partir das APIS REST do Kubernetes.
> - Faça o controlador gerar e preencher dinamicamente as requisições do tipo *Job* de que seu código precisará para rodar o scraping e concretizar a clonagem da página.
