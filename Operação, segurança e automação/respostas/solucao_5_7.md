# Solução: Exercício 5.7 - Faça o deploy para o serverless

**Guia de Solução:**

### Passo a Passo

1. **Ajuste o YAML do Ping-pong**: Altere o atributo `kind: Deployment` para `kind: Service` (do `apiVersion: serving.knative.dev/v1`). Especifique o template da imagem dentro dessa estrutura. O Knative cuidará automaticamente dos *Pods*, autoescalonamento e tráfego.
2. **Ajustar as Conexões do Cluster**: O *Log output*, que dependia do ping-pong, deve ser atualizado para contatar o serviço usando o Nome de Domínio Totalmente Qualificado do Kubernetes (*FQDN*), por exemplo: `http://pingpong.default.svc.cluster.local`.
3. **Criação do HTTPRoute (Opcional)**: Se quiser testar chamadas originadas fora do cluster, crie o filtro `URLRewrite` apontando o `hostname` para a URL dinâmica do Knative. Por exemplo, substituindo a raiz usando `replacePrefixMatch`. 
4. **Apply**: Faça o deploy dessa configuração; ao deixar o serviço de Log sem requisitar o Ping-pong, ele escalará o ping-pong para zero *pods*. No momento em que uma chamada HTTP for realizada, ele rapidamente criará um novo.
