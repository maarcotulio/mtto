---
title: "Como uma LLM vira um agente"
date: 2026-09-01
summary: "O último post da série: ReAct, uso de ferramentas, memória, reflexão, MCP e harness. O que transforma uma LLM em um agente, mais um guia prático de como eu uso IA no trabalho e para aprender."
tags: ["AI", "Work", "Agents", "LLM"]
---

Antes de entrar no tópico, vale dizer que este é o último post da série. Recomendo ler os anteriores para ter mais contexto. A diferença real entre uma LLM e um agente, é dar um cinto de utilidades do Batman ao modelo, e isso aumenta a corretude das respostas. Você já deve ter visto, ao usar os agentes, que dá para configurar o esforço que ele vai empregar em cada resposta. Mas o que exatamente isso faz?

## ReAct

ReAct é o método apresentado no paper de mesmo nome, e o nome é a junção de *Reason* e *Act* (raciocinar e agir). A ideia é dar à LLM uma estrutura para seguir, com o objetivo de melhorar as respostas. Basicamente, o que eles propõem é o seguinte ciclo:

```text
pensamento → ação → observação → novo pensamento
```

A maioria dos modelos de raciocínio hoje deixa você ver pelo menos um resumo da linha de raciocínio: essa é a parte do pensamento. A ação varia, o modelo pode querer chamar uma ferramenta ou editar um arquivo. A observação é verificar se o resultado foi o que se esperava. E a LLM repete esse ciclo por várias iterações, num loop autônomo.

{{< mermaid >}}
flowchart LR
    P[Pensamento] --> A[Ação]
    A --> O[Observação]
    O --> P
{{</ mermaid >}}

Isso provou ajudar bastante em um problema específico: a alucinação. A alucinação acontece quando o modelo gera uma informação que parece plausível, mas é falsa ou inventada. Como o ReAct obriga o modelo a checar cada passo contra uma observação real vinda de uma fonte externa, ele tem menos espaço para inventar.

![Comparação entre responder direto, só raciocinar (CoT), só agir e ReAct em uma pergunta do HotpotQA e em uma tarefa do AlfWorld: apenas o ReAct, intercalando pensamento e ação, chega à resposta certa](/images/posts/llm_para_agente/react-teaser.png)

*Figura 1 de Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), [arXiv:2210.03629](https://arxiv.org/abs/2210.03629), sob licença [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

Ligando isso ao esforço que a gente escolhe nos agentes, em termos bem simples, aumentar o esforço é como dizer:

> Faça vários ciclos de pensamento e ação antes de responder, para garantir que a resposta seja precisa.

Mais precisamente, o esforço controla quanto o modelo "pensa" (quantos tokens de raciocínio ele gasta) antes de agir.

![Seletor de esforço do Claude Code, com uma barra de "Faster" a "Smarter" e as opções low, medium, high, xhigh e max](/images/posts/llm_para_agente/effort.png)

Como você prestou atenção na parte de ação, e eu falei sobre usar ferramentas, vamos aprofundar nisso.

## Uso de ferramentas

O provedor pode permitir que o modelo use ferramentas, e esse recurso é chamado de function calling. Alguns modelos já vêm com ferramentas prontas, mas você também pode expor as suas.

Vamos explicar com exemplos. Suas ferramentas podem ser o acesso ao terminal. Se você já usou um CLI como o Claude Code ou o Codex, provavelmente se lembra dele pedindo permissão por causa de um comando: isso acontece porque ele quer usar uma ferramenta.

![Claude Code pedindo permissão para criar um arquivo, com as opções "Yes", "Yes, and switch to accept edits" e "No"](/images/posts/llm_para_agente/request-permission.png)

Além das suas ferramentas, o provedor também oferece as dele. Veja o trecho a seguir, tirado do [site da Anthropic](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview).

```ts
const client = new Anthropic();
const response = await client.messages.create({
  model: "claude-opus-5",
  max_tokens: 1024,
  tools: [{ type: "web_search_20260209", name: "web_search" }],
  messages: [{ role: "user", content: "What's the latest on the Mars rover?" }]
});
console.log(response.content);
```

Nesse caso, estamos usando a ferramenta de busca da Anthropic. Existem várias outras, todas detalhadas no site.

## Memória

Aqui vale uma distinção. O contexto de uma conversa é uma memória de curto prazo: quando você troca de sessão, ela acaba. De forma simples, todas as mensagens anteriores são inseridas no contexto de entrada, e o modelo aplica o mecanismo de atenção sobre todos esses tokens.

A memória de longo prazo é algo que persiste fora da janela de contexto. Você provavelmente já viu isso conversando no site do ChatGPT ou do Claude, quando ele guarda algumas informações importantes. Esses dados são recuperados sempre que têm relação com o que você falou e inseridos antes de a mensagem chegar na entrada do modelo.

![Painel "Memory summary" do Claude mostrando preferências guardadas sobre o usuário, com seções de visão geral e preferências](/images/posts/llm_para_agente/long-memory.png)

## Reflexão

Outra melhoria, proposta depois do ReAct, foi a de fazer a LLM guardar o feedback dos erros que ela cometeu. Essa ideia vem do paper Reflexion. Por exemplo: estamos escrevendo um código em C e ela esqueceu o famoso `;`. Ao compilar, o output aponta o erro. Esse é o feedback: a partir dele, o modelo analisa o que deu errado e guarda a lição para as próximas tentativas.

## MCP (Model Context Protocol)

É um protocolo aberto, criado pela Anthropic, que permite conectar o modelo a diferentes ferramentas, fontes de dados e serviços externos. Dessa forma, você amplia as ações que o agente pode executar. Por exemplo, o MCP do GitHub permite que o agente crie PRs e issues por você.

{{< mermaid >}}
flowchart TD
    Host([MCP Host]) --> Client[Client]
    
    Client --> Server1[[MCP Server]]
    Client --> Server2[[MCP Server]]
    
    Server1 -.-> Github[/"GitHub<br>tools<br>resources"/]
    Server2 -.-> Database[("Database<br>tools<br>resources")]
{{</ mermaid >}}


## Harness

O harness é a infraestrutura que envolve e controla o modelo. Ele define regras, ferramentas, memória, permissões, validações e muito mais.

Simplificando: é tudo que vimos neste post somado a uma LLM. É isso que transforma o modelo em um agente capaz de executar uma tarefa.

{{< mermaid >}}
flowchart LR
    U[Usuário] --> C[contexto]
    C --> L

    subgraph Harness
        direction TB
        L[LLM] --> T[tools]
        T --> E[execução]
        E --> F[feedback]
    end
{{</ mermaid >}}

## Como utilizar a IA no trabalho

Existem vários guias na internet, e até cursos, sobre como usar a IA. Eu pesquisei bastante tentando encontrar a melhor forma, e a verdade é que não existe uma forma correta globalmente aceita. Existem algumas recomendações, com base nos usos que já tive e nas leituras que fiz de posts que dão conselhos.

### Codex vs Claude Code

O que muda entre eles não é só o modelo base, é como cada um foi pós-treinado pra usar ferramentas. Para quem não sabe, existem duas ferramentas principais: o Claude Code, da Anthropic, e o Codex, da OpenAI. Grande parte das discussões gira em torno desses dois, mesmo que existam outros, como Kimi, Grok e Gemini. Vale testar como eles se saem para programar. A resposta para qual dos dois escolher depende da preferência, já que os trade-offs são diferentes. Eu recomendaria testar os dois e depois decidir. De maneira geral, minhas impressões foram que o código do Codex é um pouco inferior ao do Claude, ele responde com muito texto e tem uma janela de contexto menor. Em compensação, ele consome menos tokens e, de vez em quando, libera um reset do limite semanal na hora que você quiser. Eu gosto mais dos resultados do Claude, seja em conversas no site, seja em código, além de a janela de contexto ser bem maior. Essas foram minhas impressões pagando o plano de US$ 20.

Recentemente comecei a usar o Gemini e sinto que as respostas são piores, mesmo usando o melhor modelo com esforço alto. Me deparei várias vezes com ele seguindo um raciocínio errado, algo que não vi com tanta frequência nos outros.

Um conselho que vale para qualquer um dos dois: não parta sempre para o modelo de ponta no esforço máximo, para não usar uma bazuca pra matar uma formiga.

| Tipo de Tarefa | Claude | ChatGPT | Por quê |
|---|---|---|---|
| Exploração/busca | Haiku | GPT-5.4  | Rápido, barato, bom o suficiente para encontrar arquivos |
| Edições simples | Haiku | GPT-5.4  | Mudanças em um único arquivo, instruções claras |
| Implementação multi-arquivo | Sonnet | GPT-5.5 | Melhor equilíbrio para programação |
| Arquitetura complexa | Opus | GPT-5.6 Sol | Precisa de raciocínio profundo |
| Revisão de PR | Sonnet | GPT-5.4 | Entende contexto, percebe nuances |
| Análise de segurança | Opus | GPT-5.6 Sol | Não pode se dar ao luxo de perder vulnerabilidades |
| Escrita de documentação | Haiku | GPT-5.4  | Estrutura é simples |
| Debug de bugs complexos | Opus | GPT-5.6 Sol | Precisa manter todo o sistema em mente |

> **Nota:** a linha de modelos da OpenAI muda com muita frequência, e a tabela reflete o panorama de agosto de 2026. Ela é baseada na tabela do long-form guide do ECC, com os modelos do ChatGPT adicionados.

### Skills e Subagents

Antes de começar, deixa eu definir os conceitos. Skill costuma ser um arquivo que define regras e passos para o agente seguir ao executar determinada tarefa. Subagents são novas instâncias do modelo criadas pelo próprio agente.

O uso de skills é muito útil. Recomendo procurar algumas que façam sentido no seu trabalho, e uma que não vale a pena ignorar é a de `TDD` (test-driven development), explico o motivo mais para frente. No repositório do ECC existe um cardápio grande de skills. Não baixe todas, já que existe um orçamento de contexto pra elas. Veja quais compensam para você.

Minha experiência com subagents se resume a alguns projetinhos. Eu, particularmente, uso muito pouco, só quando a tarefa dá para dividir bem. No geral, acho melhor ter apenas um agente. Sei que existem opções para conversar com um agente que delega para vários, como o [firstmate](https://github.com/kunchenguid/firstmate), mas mesmo assim prefiro não usar isso no trabalho. Outro tópico importante são os MCPs, que permitem ao agente se conectar com outros serviços. É muito útil, mas mantenho uma lista pequena.

### Como se atualizar

Se você tinha decidido, por raiva dos modelos, que não ia estudar porque isso seria inútil, mas agora quer se atualizar de verdade, porque começou a perceber que a IA vai fazer parte do nosso dia a dia daqui para frente, existem algumas recomendações. Inclusive, para escrever este post, eu li o conteúdo delas para embasar melhor. Começo pelos guias escritos por Affaan Mustafa, criador do [ECC](https://github.com/affaan-m/ECC/tree/main). O ECC é um harness de agentes, mais focado no Claude Code, mesmo suportando outros LLMs. Nesse projeto existem guias que cobrem vários temas, como MCP, otimização de tokens, skills, hooks e subagents. Existe também o blog do Akita, de que gostei bastante, com este [post](https://akitaonrails.com/2026/02/20/do-zero-a-pos-producao-em-1-semana-como-usar-ia-em-projetos-de-verdade-bastidores-do-the-m-akita-chronicles/), onde ele explica as lições que aprendeu usando essas ferramentas. Esses são bons guias para começar, o resto fica por sua conta de pesquisar. Mesmo que eu explique alguns conceitos neste post, para se aprofundar de verdade é preciso ler mais.

### Janela de Contexto

Com as skills escolhidas e o modelo decidido, vamos para a parte de contexto. Toda vez que você abre uma sessão com o seu agente, existe o contexto, e quanto mais você conversa e o agente trabalha, mais ele cresce. Tanto o Claude quanto o Codex têm compressão de contexto. Por um lado é uma maravilha, mas, como sempre, vem com um trade-off: o agente não lembra tão bem das coisas, e eu sinto isso claramente. Por isso, recomendo iniciar uma nova sessão. Fique de olho no seu contexto, e se perceber que já está quase cheio, troque de sessão.

![Saída do comando /context no Claude Code, com a divisão do uso de tokens entre system prompt, ferramentas, skills, mensagens e espaço livre](/images/posts/llm_para_agente/context.png)

### Sandbox

Já rolaram posts em redes sociais de gente que teve arquivos apagados pelo agente por um erro do modelo ([um exemplo](https://x.com/lifeofjer/article/2048103471019434248)). Para isso, é bom ter um sandbox: ele deixa o ambiente mais seguro para você usar a ferramenta. Você pode impedir que o agente acesse arquivos específicos, por exemplo o `.env`, entre muitas outras restrições, garantindo uma segurança maior. Existem diversos repositórios que implementam essa proteção, escolha o que achar melhor.

### CLAUDE.md ou AGENTS.md

Esses são documentos que a IA lê para entender melhor o projeto. É recomendado ter um, já que isso evita explicações repetitivas sobre os padrões do projeto.

### Workflow

Agora o principal: como eu de fato uso no dia a dia. Primeiro, faço um planejamento da tarefa em que estou trabalhando e leio com atenção para ver se encontro algum ponto que precise discutir com o agente para melhorar ou mudar. Depois, uso a skill de `tdd` para implementar o plano, garantindo testes de cobertura e de regressão. A IA é muito boa em criar testes e otimizações, o que dá mais confiança no projeto. Claro, sempre fique de olho para ver se ela não está criando testes desnecessários. Sinceramente, isso nunca aconteceu comigo.

Uso uma nova sessão para o papel de revisor e faço, no máximo, três revisões com a IA. Fique atento se um problema continua aparecendo entre as revisões, isso merece sua atenção. Depois disso, faço uma revisão linha a linha de tudo que foi escrito e dos arquivos, conferindo se está tudo certo. Não deixe para revisar todo o código só no final, isso cansa demais de uma vez. É melhor ler tudo sempre que for fazer um commit. Conselho: tenha um CI/CD de segurança com CodeQL, por exemplo, ou faça uma passagem de revisão com a IA.

## Como utilizo para aprender e no dia a dia

A IA pode ser um professor pessoal muito eficiente se você souber usar. Esse [vídeo](https://youtu.be/kzcI5F4tGiU?si=NihnohbgqGqsuhen) explica um método para aprender com a IA. Testei e achei bom. Fiz uma adaptação para usar com o Claude, já que o método original usa o Pi, mas ainda não publiquei porque estou testando mais.

Fugindo um pouco do tópico: acho muito bom usá-la principalmente quando quero uma resposta específica e, se eu pesquisar por conta própria, há uma grande chance de tomar spoilers de alguma obra que estou vendo ou lendo no momento.

## Referências

- Yao, S. et al. "ReAct: Synergizing Reasoning and Acting in Language Models." 2022. [arXiv:2210.03629](https://arxiv.org/abs/2210.03629)
- Shinn, N. et al. "Reflexion: Language Agents with Verbal Reinforcement Learning." NeurIPS 2023. [arXiv:2303.11366](https://arxiv.org/abs/2303.11366)
- Anthropic. "Tool use with Claude." [Documentação](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
- Anthropic. "Introducing the Model Context Protocol." 2024. [anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)
- Model Context Protocol. Especificação e documentação. [modelcontextprotocol.io](https://modelcontextprotocol.io)
- Affaan Mustafa. ECC (The Agent Harness Performance Optimization System). [github.com/affaan-m/ECC](https://github.com/affaan-m/ECC)
- Akita. "Do Zero à Pós-Produção em 1 Semana." [akitaonrails.com](https://akitaonrails.com/2026/02/20/do-zero-a-pos-producao-em-1-semana-como-usar-ia-em-projetos-de-verdade-bastidores-do-the-m-akita-chronicles/)
