# Exercício 5.6: Testando Serverless

**Instruções do Exercício:**

> Experimente os conceitos de *Serverless computing* criando sua própria infraestrutura em cima do Kubernetes.
> - Instale o componente **Knative Serving** no seu cluster k3d (lembre-se de criar o cluster desabilitando o roteador Traefik nativo para não haver conflito de portas).
> - Siga o guia oficial de instalação via YAML, escolhendo o Kourier como camada de rede e o Magic DNS para resolução local.
> - Se houver falhas e *CrashLoopBackOffs*, inspecione os logs para corrigi-los.
> - Por fim, execute e teste os exemplos de Deploy, Autoscaling (escalonamento a zero) e Traffic splitting fornecidos na documentação do Knative.
