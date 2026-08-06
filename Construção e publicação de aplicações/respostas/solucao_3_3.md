# Solução: Exercício 3.3 - Ingress com TLS

**Guia de Solução:**

### Passo a Passo

1. Crie um Service interno para a aplicação.
2. Crie um Ingress com regra de host e caminho.
3. Para TLS manual, crie um Secret do tipo `kubernetes.io/tls`.
4. Para cert-manager, adicione annotation do issuer e valide o Certificate.
