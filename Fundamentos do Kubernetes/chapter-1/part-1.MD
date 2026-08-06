# Capítulo 1: Primeiros passos


### Conteúdo
- [Primeiros passos](#capítulo-1-primeiros-passos)
  - [Pré-requisitos](#pré-requisitos) 
  - [Material do curso](#material-do-curso)
  - [Uso de LLMs](#uso-de-llms)
  - [Sobre o material](#sobre-o-material)




### Pré-requisitos
Espera-se que os participantes tenham concluído o [DevOps com Docker](https://courses.mooc.fi/org/uh-cs/courses/devops-with-docker) ou tenham experiência com Docker e Docker Compose. Além disso, é necessária experiência com desenvolvimento web, como Desenvolvimento Web Full Stack ou equivalente

### Material do curso
O material do curso deve ser lido parte por parte, do início ao fim. O material contém exercícios, que estão posicionados de forma que o conteúdo anterior forneça informações suficientes para a resolução de cada um. Faça os exercícios à medida que avança no material. Conforme você progride, precisará buscar cada vez mais informações na internet. Você também deve complementar seu conhecimento com a [documentação oficial](https://kubernetes.io/docs/home/)!


### Uso de LLMs

Grandes modelos de linguagem (LLMs), como [ChatGPT](https://chatgpt.com/) , [Claude](https://claude.ai/new) e [GitHub Copilot](https://github.com/features/copilot?locale=pt-br) , provaram ser muito úteis no desenvolvimento de software.
Pessoalmente, utilizo principalmente o Copilot, que se integra perfeitamente ao VS Code graças a um [plugin ](https://visualstudio.microsoft.com/pt-br/github-copilot/) e também se mostrou bastante útil no contexto do Docker.

Existem muitos casos de uso para os LLMs. Por exemplo, você pode:

- Pedir ao LLM por várias configurações
- Pedir ao LLM para esclarecer o que uma determinada configuração faz exatamente
- Colar mensagens de erro e configurações no LLM e perguntar onde pode estar o problema
- Pedir esclarecimentos sobre conceitos fundamentais

O grau de utilidade das dicas fornecidas pelo Copilot e por outros modelos de linguagem varia. Talvez o maior problema desses modelos seja a [alucinação](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence)): eles às vezes geram respostas que parecem muito convincentes, mas que estão completamente erradas. Ao programar, é claro que o código "alucinado" costuma ser detectado rapidamente quando não funciona. As situações mais problemáticas ocorrem quando o código ou a configuração gerada pelo modelo parece funcionar, mas contém bugs difíceis de detectar, como vulnerabilidades de segurança.

Outro problema na aplicação de modelos de linguagem ao desenvolvimento de software é a dificuldade que eles têm para "entender" projetos grandes e, por exemplo, gerar funcionalidades que exijam mudanças em vários arquivos. Atualmente, os modelos de linguagem também não conseguem generalizar o código; ou seja, se o seu código tiver funções ou componentes existentes que o modelo poderia usar com pequenas alterações, ele não fará isso. O resultado pode ser a deterioração da sua base de código, pois os modelos de linguagem geram muita repetição. Veja mais sobre isso, por exemplo, [aqui](https://visualstudiomagazine.com/articles/2024/01/25/copilot-research.aspx).

Ao usar modelos de linguagem, a responsabilidade é sempre do programador.

O rápido desenvolvimento dos modelos de linguagem coloca o aluno em uma posição desafiadora: vale a pena (ou sequer é necessário) aprender as coisas num nível detalhado quando se pode conseguir quase tudo pronto através dos modelos de linguagem?
Nesse ponto, vale a pena lembrar da velha sabedoria de [Brian Kernighan](https://en.wikipedia.org/wiki/Brian_Kernighan) , coautor do livro The C Programming Language:

"Depurar é duas vezes mais difícil do que escrever o código em primeiro lugar. Portanto, se você escreve o código da forma mais inteligente possível, você, por definição, não é inteligente o suficiente para depurá-lo." - Brian Kernighan


Em outras palavras, como a depuração é duas vezes mais difícil do que a programação em si, não vale a pena programar um código que você mal consegue entender. Como seria possível depurar algo se a programação for terceirizada para um modelo de linguagem e o desenvolvedor de software não entender absolutamente nada do código? O mesmo se aplica à configuração do Kubernetes, e pode ser até mais grave: se você não entende os fundamentos, depurar problemas complexos do Kubernetes é simplesmente impossível.


Até agora, o desenvolvimento dos modelos de linguagem e da inteligência artificial continua numa fase em que eles não são autossuficientes, deixando os problemas mais difíceis para os humanos resolverem. Devido a isso, mesmo desenvolvedores iniciantes devem aprender os fundamentos muito bem, por precaução. Pode ser que, apesar da evolução dos modelos, conhecimentos ainda mais aprofundados se façam necessários. A inteligência artificial faz as tarefas fáceis, mas os humanos precisam resolver as confusões mais complicadas causadas pela IA. O GitHub Copilot tem um nome muito apropriado: ele é um Copiloto, um segundo piloto que auxilia o piloto principal em uma aeronave. O programador ainda é o piloto principal, o capitão, e carrega a responsabilidade final.

### Sobre o material
O curso DevOps com Kubernetes foi criado pelo estimado Docker Captain [Jami Kousa](https://www.docker.com/contributors/jami-kousa/) com a ajuda do departamento de ciência da computação da Universidade de Helsinque, pela academia de desenvolvimento de software ([Toska](https://toska.dev/)).
A partir de 2024, o curso passou a ser liderado por Matti Luukkainen, o criador do Full Stack Open. Você pode contatá-lo pelo e-mail matti.luukkainen@helsinki.fi .

Este material está licenciado sob a Licença [Creative Commons BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/) , portanto você pode usá-lo e distribuí-lo livremente, desde que os criadores originais recebam os créditos. Se você fizer alterações no material e quiser distribuir a versão alterada, ela deverá ser licenciada sob a mesma licença. O uso deste material para fins comerciais é proibido sem permissão.