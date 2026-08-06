# Solução: Exercício 5.4 - Wikipedia com init e sidecar

**Guia de Solução:**

### Passo a Passo

1. **Criar Volume**: Crie um manifesto de Deployment com um volume do tipo `emptyDir` para compartilhar os dados entre os contêineres do Pod. Ele será montado em `/usr/share/nginx/html`.
2. **Init Container**: Na secção `initContainers`, configure um contêiner baseado em `curlimages/curl`. O comando deverá ser: `sh -c "curl -L -o /usr/share/nginx/html/index.html https://en.wikipedia.org/wiki/Kubernetes"`.
3. **Sidecar Container**: Na seção regular `containers`, crie um contêiner secundário usando `curlimages/curl`. Em vez de morrer, ele deve rodar um loop longo: `sh -c "while true; do sleep $((RANDOM % 600 + 300)); curl -L -o /usr/share/nginx/html/index.html https://en.wikipedia.org/wiki/Special:Random; done"`.
4. **Main Container**: Crie o contêiner final usando `nginx:alpine` com o `volumeMounts` devidamente exposto, que apenas servirá os arquivos HTML alterados de maneira contínua.
