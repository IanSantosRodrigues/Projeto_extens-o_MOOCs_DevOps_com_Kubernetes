# Solução: Exercício 4.9 - O projeto, etapa 25

**Guia de Solução:**

### Passo a Passo

1. **Criar Diretórios Kustomize**: Reestruture seus arquivos YAML para usar *bases* e *overlays*. Crie a pasta `base` com os YAMLs principais, e crie as pastas `overlays/staging` e `overlays/production`.
2. **Configurar Staging**: No diretório de `staging`, modifique o `kustomization.yaml` aplicando *patches* que alterem a URL do *webhook* do broadcaster para uma URL falsa (mock) e que excluam ou não chamem o CronJob de backup.
3. **Configurar Production**: No diretório de `production`, o *webhook* recebe a URL verdadeira e o *CronJob* de backup é incluído nas definições.
4. **Ajuste de Namespaces e Branches**: Crie duas `Applications` no ArgoCD. A de staging aponta para o path `overlays/staging` e `targetRevision: main`. A de produção aponta para `overlays/production` e pode usar um padrão de *tags* na revisão para só disparar quando houver a tag oficial.
5. **Aplicar**: Commit a estrutura. O Argo gerenciará os dois namespaces de forma distinta a partir do mesmo código-base.
