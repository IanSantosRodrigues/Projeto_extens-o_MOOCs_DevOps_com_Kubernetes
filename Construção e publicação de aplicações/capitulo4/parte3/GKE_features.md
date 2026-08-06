# Recursos de GKE, Ingress, TLS e Canary Deployment

## O que você aprenderá nesta página
- Publicar aplicações com Ingress ou Gateway
- Configurar HTTPS com TLS e cert-manager
- Usar labels, annotations e Canary Deployment para controlar tráfego

Depois que a aplicação está no cluster e o pipeline consegue publicar novas versões, o próximo passo é lidar com tráfego externo, segurança de transporte e estratégias de lançamento. Em cloud, esses temas aparecem juntos: o provedor oferece balanceadores, o Kubernetes oferece objetos declarativos e ferramentas como cert-manager automatizam certificados.

## Ingress e Ingress Controller

`Ingress` é um recurso que descreve regras HTTP e HTTPS para chegar aos Services internos. Ele não funciona sozinho: é necessário um `Ingress Controller`, como NGINX Ingress Controller, Traefik ou controlador do provedor de nuvem.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-svc
                port:
                  number: 80
```

## TLS e cert-manager

Para HTTPS, o Ingress precisa de um certificado TLS. Em estudos, esse certificado pode ser criado manualmente. Em produção, é comum usar `cert-manager` para emitir e renovar certificados automaticamente.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-tls
spec:
  secretName: app-tls-secret
  dnsNames:
    - app.example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

## Labels e annotations

Labels são usadas para seleção e organização. Annotations armazenam metadados usados por controladores, como Ingress Controller, cert-manager e ferramentas de observabilidade. Use labels para aquilo que será selecionado; use annotations para instruções auxiliares interpretadas por controladores externos.

## Canary Deployment

Canary Deployment libera uma nova versão para uma parte pequena do tráfego antes de substituir tudo. Com Ingress Controller, isso pode ser feito por annotations específicas; com ferramentas como Argo Rollouts, pode ser feito de forma mais estruturada e observável. A ideia é reduzir risco antes de ampliar o tráfego para todos os usuários.

## Fechamento do capítulo

Nesta parte, a aplicação deixou de ser apenas um Pod em execução. Ela passou a ter configuração externa, persistência, rede estável, publicação HTTP/HTTPS, pipeline e observabilidade. Esse é o caminho para transformar uma aplicação básica em uma aplicação preparada para cloud.
