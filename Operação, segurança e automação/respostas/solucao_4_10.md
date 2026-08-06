# Solução: Exercício 4.10 - O projeto, o grand finale

**Guia de Solução:**

### Passo a Passo

1. **Separação de Repositórios**: Crie um repositório chamado `meu-projeto-codigo` (com arquivos Node/Go/Python) e um repositório `meu-projeto-manifestos` (apenas arquivos YAML/Kustomize).
2. **Pipeline de CI (Código)**: Configure o GitHub Actions no repositório de código para compilar as imagens e enviá-las ao Docker Hub, anexando o `$GITHUB_SHA` como tag da imagem.
3. **Integração CI -> Manifestos**: Adicione um passo final ao GitHub Actions que faz um `git clone` do repositório `meu-projeto-manifestos` com um Token de Acesso Pessoal (PAT). Nesse diretório, rode `kustomize edit set image imagem-backend=meu-usuario/backend:$GITHUB_SHA`, e faça o *commit* e *push* da mudança no repositório de manifestos.
4. **Pipeline de CD (ArgoCD)**: Configure o ArgoCD para escutar o repositório `meu-projeto-manifestos`. A alteração do *commit* de CI acionará o *Sync* automático do ArgoCD.
