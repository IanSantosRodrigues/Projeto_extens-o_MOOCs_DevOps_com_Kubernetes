# Solução: Exercício 5.5 - Comparação de Plataformas

**Guia de Solução:**

### Passo a Passo (Resolução Teórica)

Crie uma lista simples de *bullet points* comparando, por exemplo, o **Rancher** contra o **OpenShift**.

*   **OpenShift (Red Hat)**: Projetado para o mundo corporativo (Enterprise). É altamente opinativo, exigindo padrões de segurança rigorosos (contêineres rootless obrigatórios). Contém CI/CD e Registry nativos profundamente integrados, mas possui alto custo de licenciamento.
*   **Rancher (SUSE)**: Foco total no open-source e em gerenciamento multi-cluster. Extremamente leve (é possível usar k3s e RKE em cima de qualquer infraestrutura) e evita *vendor lock-in* massivos. 
*   **Veredito de Exemplo**: O *Rancher* é "melhor" para nossa equipe fictícia porque priorizamos uma infraestrutura leve, sem dependência pesada de um ecossistema comercial rígido, permitindo a transição livre entre nuvens públicas e servidores internos mantendo as ferramentas CNCF padrão em 100% dos ambientes.
