# Solução: Exercício 5.6 - Testando Serverless

**Guia de Solução:**

### Passo a Passo

1. **Cluster k3d Personalizado**: Remova o traefik padrão durante a criação do cluster com a *flag* apropriada: `k3d cluster create --k3s-arg "--disable=traefik@server:0"`.
2. **Instalação do Knative Serving**: Aplique os manifestos do Core do Knative e CRDs. 
3. **Camada de Rede (Kourier)**: Instale os manifestos de rede focados em Kourier e aplique um patch para que o Knative o defina como *Ingress* padrão: `kubectl patch configmap/config-network --namespace knative-serving --type merge --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'`.
4. **Magic DNS**: Aplique o Job de configuração para o `sslip.io`, permitindo que os domínios internos possuam rotas válidas para os testes de *curl* locais.
5. **Troubleshooting**: Inspecione o `webhook` ou `controller` com `kubectl logs -n knative-serving` caso fiquem em estado de falha (dependendo dos recursos da máquina, aguardar muitas vezes é a solução).
6. **Testes Finais**: Crie um Knative `Service` genérico, acompanhe os pods nascendo a partir do zero a cada requisição (Scale to Zero) e explore como fazer as transições de revisões via YAML.
