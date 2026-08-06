# Solução: Exercício 5.2 - Primeiros passos com a malha de serviço Istio

**Guia de Solução:**

### Passo a Passo

1. **Instalação do CLI**: Siga o guia de documentação e baixe os binários do Istio. Ex: `curl -L https://istio.io/downloadIstio | sh -` e inclua o executável no seu PATH.
2. **Configurar k3d/Kubernetes**: Execute `istioctl install --set profile=demo -y` no cluster para instalar a estrutura do *Control Plane* do Istio.
3. **Injeção Automática**: Rotule o *namespace* padrão para usar o proxy automático do Istio: `kubectl label namespace default istio-injection=enabled` (ou as configurações específicas para a versão Ambient, sem sidecars).
4. **Deploy do Sample App**: Faça o apply dos manifestos fornecidos pelo repositório do Istio (como o *Bookinfo application*).
5. **Monitoramento e Limpeza**: Exponha o *Kiali* via `istioctl dashboard kiali`. Visualize os nós se comunicando. Em seguida, limpe as instalações conforme instruído pela documentação.
