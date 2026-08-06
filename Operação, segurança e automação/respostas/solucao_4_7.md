# Solução: Exercício 4.7 - Passos de bebê para GitOps

**Guia de Solução:**

### Passo a Passo

1. **Instalar o ArgoCD**: Execute os manifestos de instalação do ArgoCD no cluster.
2. **Organizar o Repositório**: Certifique-se de que os arquivos YAML do *Log output* estão centralizados em uma pasta no seu repositório Git, idealmente estruturados com um `kustomization.yaml`.
3. **Criar a Aplicação no ArgoCD**: Através da interface web do ArgoCD ou de um manifesto do tipo `Application`, crie a entidade apontando para o seu repositório Git e para o diretório dos manifestos.
4. **Habilitar Sync Automático**: Nas configurações do ArgoCD `Application`, mude o *Sync Policy* para `Automated`. Qualquer *commit* realizado na ramificação principal alterará automaticamente o estado dos *Pods* no Kubernetes.
