# Solução: Exercício 5.8 - O Panorama (Landscape)

**Guia de Solução:**

### Passo a Passo

1. **Acessar o Mapa**: Abra o CNCF Cloud Native Landscape (versão interativa ajuda a procurar itens via texto).
2. **Circule os Usados**: Salve a imagem/Screenshot do navegador. Num editor de imagens (Paint, Photoshop), pegue um círculo Vermelho e marque as ferramentas abordadas nas lições: **Kubernetes, Helm, ArgoCD, Prometheus, Grafana, NATS, Istio, Knative** e, caso os use profissionalmente, **Docker, GitHub Actions**, etc.
3. **Circule as Dependências**: Com um círculo Azul, marque itens base em que esses projetos rodam. Exemplo: Kubernetes depende de **etcd** para o Control Plane, **containerd/CRI-O** para execução de imagens e **CoreDNS** e **Flannel/Calico** para as camadas de rede.
4. **Lista de Contexto**: Num arquivo `.txt` simples (ou `.md`), relacione os itens, como: 
   - *Istio*: Focado no curso para Service Mesh e divisões de tráfego de redes;
   - *containerd*: Uso indireto via Docker Desktop / k3s fora do curso para empacotar bibliotecas;
   - *CoreDNS*: Uso indireto no k3d gerenciando `svc.cluster.local`.
