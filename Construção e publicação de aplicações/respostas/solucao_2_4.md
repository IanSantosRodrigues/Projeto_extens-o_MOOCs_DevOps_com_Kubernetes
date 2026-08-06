# Solução: Exercício 2.4 - Volumes persistentes

**Guia de Solução:**

### Passo a Passo

1. Crie um PVC solicitando, por exemplo, `1Gi`.
2. Monte o PVC em `/usr/share/nginx/html` ou outro caminho de teste.
3. Entre no Pod e grave um arquivo no diretório montado.
4. Recrie o Pod mantendo o PVC e confirme que o arquivo permaneceu.
