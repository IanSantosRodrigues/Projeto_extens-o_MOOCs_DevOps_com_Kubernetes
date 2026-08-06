# Exercício 5.4: Wikipedia com init e sidecar

**Instruções do Exercício:**

> Escreva um aplicativo que sirva o conteúdo de páginas da Wikipedia a partir de *containers* rodando no Kubernetes. A arquitetura do *pod* deve conter:
> - O contêiner principal baseado na imagem *nginx*, configurado simplesmente para servir qualquer conteúdo HTML presente no seu diretório público padrão (`www`).
> - Um contêiner de inicialização (*init container*) que, no início do *Pod*, execute um `curl` na página da Wikipedia sobre o Kubernetes e salve o conteúdo no diretório público montado no volume.
> - Um *sidecar container* rodando em paralelo que aguarde um tempo aleatório entre 5 e 15 minutos, execute um `curl` em uma página aleatória da Wikipedia (usando a URL de *Special:Random*) e sobrescreva o conteúdo atual do diretório do nginx.
