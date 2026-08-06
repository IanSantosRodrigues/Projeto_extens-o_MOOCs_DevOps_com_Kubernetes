# Solução: Exercício 5.1 - Faça você mesmo um CRD e Controller

**Guia de Solução:**

### Passo a Passo

1. **Definição da CRD**: Crie o YAML da CRD `DummySite`. No `openAPIV3Schema`, defina a propriedade `website_url` do tipo `string`. Aplique no cluster.
2. **Construção do Controlador**: Escreva uma aplicação em JavaScript usando a biblioteca `@kubernetes/client-node`. 
3. **Observar a API**: Faça o controlador utilizar um *Watcher* em `/apis/seugrupo/v1/dummysites`. Quando ele capturar o evento `ADDED`, leia a variável `website_url`.
4. **Criação do Job**: No mesmo instante de detecção, monte um objeto JSON de `Job`. A especificação do contêiner deste Job deve rodar uma imagem simples (como Alpine ou cURL) executando `sh -c "curl -L $WEBSITE_URL > /usr/share/nginx/html/index.html"` ou logicamente equivalente. Use as chamadas POST para persistir o Job no Kubernetes.
