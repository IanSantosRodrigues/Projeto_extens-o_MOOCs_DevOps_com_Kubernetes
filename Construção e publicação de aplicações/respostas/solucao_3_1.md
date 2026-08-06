# Solução: Exercício 3.1 - Aplicação preparada para cloud

**Guia de Solução:**

### Passo a Passo

1. Garanta que a imagem tenha tag explícita e não dependa de `latest`.
2. Separe configuração em ConfigMaps e Secrets.
3. Declare Service, probes e recursos mínimos no manifesto.
4. Valide reaplicando os manifestos em um namespace limpo.
