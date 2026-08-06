# Solução: Exercício 5.3 - Log app, a Edição Service Mesh

**Guia de Solução:**

### Passo a Passo

1. **Criar a Aplicação Greeter**: Escreva uma micro-API (greeter) que retorna 'Hello'. Construa duas versões da imagem Docker (v1 e v2) com uma leve diferença no retorno para diferencia-las.
2. **Deployments**: Crie dois `Deployments` no cluster, um usando a v1 e outro a v2, com labels `version: v1` e `version: v2`.
3. **VirtualService/HTTPRoute**: Em vez de usar balanceamento Kubernetes tradicional, crie um recurso `VirtualService` do Istio acoplado a um `Gateway` ou um recurso `HTTPRoute` apontando para os serviços *greeter-svc-1* e *greeter-svc-2*.
4. **Regras de Pesos (Weights)**: Na definição das rotas, atribua `weight: 75` para o destino v1 e `weight: 25` para o destino v2. O Log app precisará chamar apenas a rota virtual. Verifique a distribuição no Kiali.
