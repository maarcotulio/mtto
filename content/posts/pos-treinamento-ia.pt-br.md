---
title: "Pós-treinamento da IA"
date: 2026-08-22
summary: "Explicando o que acontece depois do pré-treinamento: SFT, Reward Model e Reinforcement Learning, os passos que transformam o modelo em um assistente alinhado."
tags: ["AI", "Work"]
---

No post anterior eu havia explicado sobre a parte do pré-treinamento, sendo a mais demorada e cara de todo o processo. Falei sobre o Transformer, a arquitetura usada pelos modelos atuais, mas não disse sobre a outra parte importante que compõe todo o processo de criar um agente. Com o que vimos até aqui, temos um modelo capaz de completar texto de forma muito eficiente, mas que ainda não sabe gerar uma resposta bem formatada ou com alguma personalidade.

## Supervised Fine-Tuning (SFT)

Depois do modelo já ter sido treinado por quantidades absurdas de dados de diferentes tópicos, agora precisamos ensinar a ela a gerar uma resposta mais satisfatória em uma conversa. Com isso, nessa primeira parte, são fornecidas várias conversas de exemplo: perguntas do usuário e como deveria ser a resposta.

Por exemplo:
```json
{
  "messages": [
    {
      "role": "system",
      "content": "Você é um assistente técnico prestativo e conciso."
    },
    {
      "role": "user",
      "content": "Como posso verificar portas abertas no Linux usando o terminal?"
    },
    {
      "role": "assistant",
      "content": "Você pode usar o comando `ss` ou o `netstat` com privilégios de superusuário:\n\n* **Com `ss` (recomendado):**\n  ```bash\n  sudo ss -tulpn\n  ```\n* **Com `netstat`:**\n  ```bash\n  sudo netstat -tulpn\n  ```\n\n**O que significam as flags:**\n* `-t`: Conexões TCP\n* `-u`: Conexões UDP\n* `-l`: Apenas portas em escuta (*listening*)\n* `-p`: Mostra o processo/PID responsável\n* `-n`: Mostra números de porta em vez de nomes de serviço"
    }
  ]
}
```

O método para esta parte do treinamento segue uma linha bem similar à forma que aprendemos no post anterior.

> Então por que esse processo é separado?

Cada uma dessas partes do treinamento resolve objetivos distintos. Caso tentasse fazer tudo de uma vez, a chance de perder qualidade em alguma parte seria grande. No pré-treinamento é criada a base: precisa de muitos dados, é aqui que a IA ganha o conhecimento. Já o SFT exige mais qualidade e faz com que o modelo consiga gerar um texto com formato e estilo mais adequados.

## Reward Model

Nesta etapa é treinada uma rede neural separada para ser um crítico das respostas, o Reward Model (RM). Ele não gera texto; recebe um prompt e uma resposta e devolve um número dizendo o quão boa aquela resposta é.

Para treinar esse crítico, pega-se o modelo já ajustado pelo SFT e geram-se, para o mesmo prompt, duas ou mais respostas diferentes. Um humano então compara essas respostas e diz qual prefere: não precisa dar uma nota exata, só apontar "essa aqui é melhor que aquela". Com muitos desses pares de comparação, o Reward Model aprende a prever uma pontuação que tenta emular esse julgamento humano.

## Reinforcement Learning (RL)

{{< katex >}}

Agora chega a parte onde o modelo (aqui chamado de `policy`) gera respostas para um prompt, o Reward Model dá uma nota para essa resposta, e essa nota vira o sinal usado para atualizar os pesos do modelo, sem precisar de mais nenhum humano no loop nesse momento. O método usado pela OpenAI no paper do InstructGPT (o mesmo pipeline usado para treinar o ChatGPT) para fazer essa atualização se chama **PPO** (Proximal Policy Optimization).

Só que existe um problema: se o objetivo for simplesmente "maximizar a nota do Reward Model", o modelo pode achar um jeito de enganar o crítico, gerando respostas que pontuam alto mas que na prática são ruins ou nem fazem sentido. Isso é chamado de **reward hacking**. Para evitar isso, o objetivo real usado no treinamento também penaliza o quanto a nova política se afastou do modelo SFT original:

$$
\max_{\pi} \; \mathbb{E}_{x,\, y \sim \pi}\big[r(x, y)\big] \;-\; \beta \, D_{KL}\big(\pi \,\|\, \pi_{ref}\big)
$$

Onde:
- $r(x, y)$: nota dada pelo Reward Model para a resposta $y$ ao prompt $x$.
- $\pi$: a política atual, o modelo sendo treinado.
- $\pi_{ref}$: o modelo SFT, usado como referência.
- $D_{KL}$: a divergência KL, uma medida de quão diferente $\pi$ ficou de $\pi_{ref}$.
- $\beta$: controla o peso dessa distância contra a recompensa.

Em outras palavras: o modelo é livre para melhorar buscando notas mais altas, mas é "puxado de volta" sempre que tenta se afastar demais do comportamento que já tinha aprendido no SFT.

![Diagrama das três etapas do RLHF: Supervised Fine-Tuning, treinamento do Reward Model e otimização da política com PPO](/images/posts/postreino/image.png)
> Imagem tirada da [AWS](https://aws.amazon.com/pt/what-is/reinforcement-learning-from-human-feedback/)

Hoje em dia também é comum ver modelos ajustados com **DPO** (Direct Preference Optimization), uma alternativa mais recente e mais simples: ela consegue o mesmo efeito de alinhar o modelo às preferências humanas sem precisar treinar um Reward Model separado nem rodar PPO, usando diretamente os pares de respostas preferida/não preferida na própria função de perda.

No fim, SFT, Reward Model e RL (seja com PPO ou com DPO) são as três peças que pegam aquele modelo que só sabia completar texto e transformam ele num assistente que responde de forma útil e alinhada com o que a gente espera. Ainda falta uma parte pra virar um agente de verdade, capaz de usar ferramentas e agir sozinho, mas essa fica pro próximo post.

## Referências

- https://arxiv.org/abs/2203.02155 (InstructGPT: origina o pipeline SFT → Reward Model → PPO)
- https://arxiv.org/abs/1707.06347 (PPO)
- https://arxiv.org/abs/2305.18290 (DPO)
- https://huggingface.co/blog/rlhf
- https://aws.amazon.com/pt/what-is/reinforcement-learning-from-human-feedback/
- https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf
- https://aws.amazon.com/pt/what-is/reinforcement-learning/
- https://arxiv.org/pdf/2204.05862
