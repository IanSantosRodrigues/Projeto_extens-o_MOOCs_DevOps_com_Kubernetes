# Solução: Exercício 2.1 - Services e comunicação interna

**Guia de Solução:**

### Passo a Passo

1. Crie um Deployment com label `app: web` e três réplicas.
2. Crie um Service ClusterIP selecionando `app: web`.
3. Use `kubectl run curl --rm -it --image=curlimages/curl -- sh` e acesse `http://web-svc`.
4. Valide com `kubectl get endpoints web-svc` se o Service encontrou os Pods.
