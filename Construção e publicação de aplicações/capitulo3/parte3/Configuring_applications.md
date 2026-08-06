# Configuração de Aplicações com ConfigMaps e Secrets

## O que você aprenderá nesta página
- Separar configuração da imagem da aplicação
- Usar ConfigMaps para dados não sensíveis
- Usar Secrets para dados sensíveis e credenciais

Uma imagem de contêiner deve ser reutilizável. Se cada ambiente exigir uma imagem diferente apenas para mudar uma URL, uma porta ou uma string de configuração, o processo de publicação fica frágil. O Kubernetes resolve esse problema com recursos declarativos para configuração.

`ConfigMap` armazena dados de configuração não sensíveis. `Secret` armazena informações sensíveis, como senhas, tokens e certificados. Ambos podem ser consumidos como variáveis de ambiente ou arquivos montados dentro do contêiner.

## ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  API_URL: http://api-svc:3000
```

A aplicação pode consumir essas chaves como variáveis de ambiente. Também é possível montar o ConfigMap como volume, útil para arquivos como `nginx.conf`, `application.yaml` ou configurações de feature flags.

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

## Secrets

Secrets têm propósito diferente: guardar dados que não devem aparecer no manifesto principal da aplicação. Por padrão, os valores são codificados em Base64, o que não é criptografia. Em clusters reais, é necessário cuidar de RBAC, criptografia em repouso e integração com gerenciadores externos de segredo quando disponível.

```bash
echo -n 'senha-forte' | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  POSTGRES_PASSWORD: c2VuaGEtZm9ydGU=
```

Uso no Deployment:

```yaml
env:
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: POSTGRES_PASSWORD
```

## Quando usar cada um

Use ConfigMap para URL de serviço, modo de execução e configurações não sensíveis. Use Secret para senha de banco, token de API, credencial de registry, certificado TLS e chaves privadas. Não coloque senha direto no Deployment e não coloque dado sensível em ConfigMap.
