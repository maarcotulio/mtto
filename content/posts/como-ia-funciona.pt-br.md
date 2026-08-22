---
title: "Como uma LLM realmente funciona?"
date: 2026-08-15
summary: "Aprofundando sobre como a IA funciona."
tags: ["AI", "Work"]
---

É inegável que o lançamento do ChatGPT, em novembro de 2022, teve um grande impacto, e grandes perguntas começaram a surgir a partir disso. Depois vieram modelos que geravam imagens, gerando ainda mais perguntas éticas sobre o uso da IA. Por fim, recentemente, com as melhorias dos LLMs, foi possível começar a utilizá-los para programação. Meu objetivo com essa série de posts é montar posts aprofundados em vários tópicos, para quem não sabe de quase nada até chegar aos usos atuais. Nesse primeiro post dessa série, vou explicar como funcionam as LLMs.

## Sobre LLMs

LLMs (Large Language Models) são modelos treinados com uma massiva quantidade de dados em grandes data centers. A qualidade do modelo depende da qualidade dos dados que serão usados no treinamento. Essa etapa inicial de treinamento tem um nome: **pré-treinamento**. É a fase mais cara e demorada de todo o processo, e é ela que vou explicar neste post: o suficiente para o modelo aprender a completar texto, mas ainda não pra virar um assistente que conversa direito (isso fica pra um próximo post).

![Mostra a imagem do data center de Memphis da xAI](/images/posts/llms/data-center.png)
> O centro de dados da xAI em Memphis. Foto: Steve Jones/Flight by Southwings para o Southern Environmental Law Center

Os modelos conseguem reconhecer e gerar textos. Existem diferentes arquiteturas para esses modelos, e o tipo de rede que será tratado aqui, que grandes modelos como ChatGPT e Claude utilizam, é chamado de Transformer. O que mudou de modelos anteriores é que agora conseguem processar os tokens de uma sequência em paralelo e decidir quais devem influenciar os outros, por meio do mecanismo da atenção. A arquitetura foi apresentada em 2017 por pesquisadores do Google no artigo [“Attention Is All You Need”](https://arxiv.org/abs/1706.03762), que teve enorme impacto no desenvolvimento da área de inteligência artificial.

## Deep Learning

> "O deep learning é um subconjunto do aprendizado de máquina orientado por redes neurais de várias camadas cujo design é inspirado na estrutura do cérebro humano. Modelos de deep learning servem de base para a maioria das inteligências artificial (IA) de última geração atualmente, desde computer vision e IA generativa até carros autônomos e robótica."
> [Explicação da IBM sobre o termo](https://www.ibm.com/br-pt/think/topics/deep-learning)

### Redes Neurais

Redes neurais, como o nome indica, são **inspiradas** no cérebro. Atente-se ao "inspiradas". A rede é dividida em nós que se ligam, esses nós são divididos em camadas. 
- Camada de entrada: que recebe os dados. 
- Camadas ocultas ou intermediárias: elas processam a informação.
- Camada de saída: fornece a resposta. 

O "neurônio" contém um valor chamado de ativação. E cada ligação ao neurônio (exceto as da camada inicial) tem um peso, que é ajustado na fase de treinamento. Esses pesos são essenciais para garantir que o modelo chegue na resposta correta.

{{< mermaid >}}
flowchart LR
    subgraph L1["Camada inicial"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Próxima camada"]
        H1(("H₁<br/>a = 0.735"))
    end

    N1 -->|"w₁ = 0.50"| H1
    N2 -->|"w₂ = -0.30"| H1
    N3 -->|"w₃ = 0.80"| H1
    N4 -->|"w₄ = 0.10"| H1
{{< /mermaid >}}

Quando você faz o uso do modelo, seja ele ChatGPT ou Claude, os pesos já estão fixos.

### Função de ativação

Se você percebeu, todo número de ativação está entre 0 e 1. Isso acontece pela função de ativação que foi utilizada; pra cada neurônio, dá pra fazer um cálculo do valor da ativação. A que foi utilizada foi a de Sigmoid.

$$ 
 \sigma(x) = \frac{1}{1 + e^{-x}}
$$

Cada neurônio, como você viu antes, tem neurônios que apontam para ele, cada um com seu respectivo valor, e os pesos. Podemos juntar todas essas informações e montar a fórmula para descobrir o valor de ativação.

$$
a_{\text{out}}=\sigma\left(\sum_{i=1}^{n} a_i w_i+b\right)=\frac{1}{1+e^{-\left(\sum_{i=1}^{n} a_i w_i+b\right)}}
$$

Calculando o valor que estava em cada neurônio no gráfico mermaid de exemplo:

$$
a_{H_1}=\sigma\left(0{,}8\cdot0{,}5+0{,}4\cdot(-0{,}3)+0{,}9\cdot0{,}8+0{,}2\cdot0{,}1\right)\approx0{,}735
$$

Existem outros métodos que são mais complexos que este e que não se prendem à faixa entre 0 e 1. Eu não irei explicá-los aqui. Cabe ao leitor, caso tenha interesse, pesquisar por mais conteúdo. Essa função de ativação não é utilizada pelos modelos de ponta; ela está aqui só pra ajudar a entender uma parte do processo que a LLM faz.

Dentro da função de ativação podemos colocar um `bias`, que vai determinar quando a soma dos pesos e ativação deve ser relevante. Em outras palavras, você ajusta o valor de partida.

$$
a_{\text{out}}=\sigma\left(\sum_{i=1}^{n}a_iw_i+b\right)
$$

### Gradient Descent

Ok, chegamos a um ponto bacana. Aqui já conseguimos fazer nosso pequeno Frankenstein, tomar algum nível de decisão. Só que temos um problema: ter que ajustar manualmente cada um desses pesos é uma tarefa inviável. O exemplo que fiz foi com apenas **um**, agora coloque valores nesse exemplo aqui e faça as contas.

{{<mermaid>}}
graph LR
    subgraph Entrada["Camada de Entrada"]
        I1((x1))
        I2((x2))
        I3((x3))
    end

    subgraph Oculta1["Camada Oculta 1"]
        H1((h1))
        H2((h2))
        H3((h3))
        H4((h4))
    end

    subgraph Oculta2["Camada Oculta 2"]
        H5((h5))
        H6((h6))
        H7((h7))
    end

    subgraph Saida["Camada de Saída"]
        O1((y1))
        O2((y2))
    end

    I1 --> H1
    I1 --> H2
    I1 --> H3
    I1 --> H4
    I2 --> H1
    I2 --> H2
    I2 --> H3
    I2 --> H4
    I3 --> H1
    I3 --> H2
    I3 --> H3
    I3 --> H4

    H1 --> H5
    H1 --> H6
    H1 --> H7
    H2 --> H5
    H2 --> H6
    H2 --> H7
    H3 --> H5
    H3 --> H6
    H3 --> H7
    H4 --> H5
    H4 --> H6
    H4 --> H7

    H5 --> O1
    H5 --> O2
    H6 --> O1
    H6 --> O2
    H7 --> O1
    H7 --> O2

    style Entrada fill:none,stroke:#333
    style Oculta1 fill:none,stroke:#333
    style Oculta2 fill:none,stroke:#333
    style Saida fill:none,stroke:#333
{{</mermaid>}}

Percebe que não faz sentido, precisaríamos de uma forma de fazer isso de maneira mais simples. Eu digo qual era a resposta esperada e ele se ajusta sozinho. Para conseguirmos fazer isso tudo, preciso primeiro ensinar o mecanismo que vai ajustar os pesos.

Vamos relembrar em termos simples o que significa gradiente: ele mostra em que direção a função cresce mais rapidamente. O que nós queremos é minimizar o erro, então precisamos ir no sentido contrário. 

$$w_{novo} = w_{atual} - \eta \cdot gradiente$$

Onde:
- $w$: é o peso da rede neural
- $\eta$: a taxa de aprendizado

Os valores para a taxa de aprendizado devem ser escolhidos com cuidado: valores altos podem passar do ponto ideal e oscilar muito, valores baixos podem fazer com que demore demais para chegar lá.

### Backpropagation

Na fórmula anterior existe o termo `gradiente`: o método usado pra calcular esse gradiente é o backpropagation. 
- Backpropagation = calcula os gradientes, as inclinações de cada peso.
- Gradient descent = usa esses gradientes para atualizar os pesos.

$$\dfrac{\partial L}{\partial w}$$

A derivada parcial da função de erro $L$ em relação ao peso $w$. Existem diferentes métodos para calcular a função de Loss; irei optar pela **Binary Cross-Entropy**.

### Exemplo

Vamos agora juntar tudo aprendido. O processo de treinamento consiste em fazer o seguinte:

```text
Entrada → Forward Pass → Backpropagation → Gradient Descent → Novos Pesos
```

Esse processo acontece repetidamente, milhares ou bilhões de vezes em modelos grandes. Para que fique claro o que é o Forward Pass: é testar colocando valores de entrada e observar o resultado de saída. 

Vamos utilizar a rede vista anteriormente.
{{< mermaid >}}
flowchart LR
    subgraph L1["Camada inicial"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Próxima camada"]
        H1(("H₁<br/>a = 0.735"))
    end

    N1 -->|"w₁ = 0.50"| H1
    N2 -->|"w₂ = -0.30"| H1
    N3 -->|"w₃ = 0.80"| H1
    N4 -->|"w₄ = 0.10"| H1
{{< /mermaid >}}

$$
a_1=0{,}8,\quad a_2=0{,}4,\quad a_3=0{,}9,\quad a_4=0{,}2
$$

Os pesos são:
$$
w_1=0{,}5,\quad w_2=-0{,}3,\quad w_3=0{,}8,\quad w_4=0{,}1
$$

E iremos definir como a resposta correta para a saída:
$$a_{\text{out}}=1$$

#### Forward Pass

Já fizemos isso anteriormente, e o cálculo deu o seguinte resultado:
$$
a_{\text{out}}=\sigma\left(0{,}8\cdot0{,}5+0{,}4\cdot(-0{,}3)+0{,}9\cdot0{,}8+0{,}2\cdot0{,}1\right)\approx0{,}735
$$

#### Backpropagation

Com esse valor faremos a conta da derivada parcial.
$$
\frac{\partial L}{\partial w_1}
$$

Para **sigmoid e binary cross-entropy**, acontece uma simplificação muito conveniente:

$$
\frac{\partial L}{\partial z}=\hat y-y
$$

Onde:
- $\hat y$: valor encontrado.
- $y$: valor desejado.

Então:

$$
\frac{\partial L}{\partial z}=0{,}735-1
$$

$$
\frac{\partial L}{\partial z}=-0{,}265
$$

Agora:

$$
z=a_1w_1+a_2w_2+\cdots
$$

portanto:

$$
\frac{\partial z}{\partial w_1}=a_1
$$

Como:

$$
a_1=0{,}8
$$

temos:

$$
\frac{\partial L}{\partial w_1} = (-0{,}265)(0{,}8)
$$

Chegamos no seguinte resultado:

$$
\frac{\partial L}{\partial w_1}\approx-0{,}212
$$

#### Gradient Descent

Vou escolher um $\eta=0{,}1$:
$$
w_1^{novo} = 0{,}5-(0{,}1)(-0{,}212)
$$

$$
w_1^{novo}\approx0{,}5212
$$

#### Forward novamente

Agora usando:

$$
w_1=0{,}5212
$$

temos:

$$
z=(0{,}8)(0{,}5212)+(0{,}4)(-0{,}3)+(0{,}9)(0{,}8)+(0{,}2)(0{,}1)
$$

$$
z\approx1{,}037
$$

Aplicando sigmoid:

$$
\hat y=\sigma(1{,}037)
$$

$$
\hat y\approx0{,}738
$$

Antes:

$$
\hat y=0{,}735
$$

Depois:

$$
\hat y=0{,}738
$$

Como nosso alvo era:

$$
y=1
$$

a previsão foi na direção correta.

E a loss também caiu:

$$
0{,}308\rightarrow0{,}303
$$

Portanto, **uma única atualização já tornou o modelo ligeiramente melhor nesse exemplo**. Lembre-se, isso foi em apenas **um** neurônio. Se fizéssemos isso com todos os outros pesos, chegaríamos em algo assim:

{{< mermaid >}}
flowchart LR

    subgraph L1["Camada inicial"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Próxima camada"]
        H1(("H₁<br/>z = 1.064<br/>a = 0.743"))
    end

    N1 -->|"w₁ = 0.5212"| H1
    N2 -->|"w₂ = -0.2894"| H1
    N3 -->|"w₃ = 0.8239"| H1
    N4 -->|"w₄ = 0.1053"| H1
{{</ mermaid >}}

## Atualidade dos Modelos

Esse modelo explicado anteriormente é uma base; nos modelos atuais, algumas coisas mudaram e ficaram mais sofisticadas. Vou comentar sobre a arquitetura mais utilizada atualmente, que é o Transformer, utilizado com frequência nos grandes modelos. É a arquitetura do paper que já mencionei lá em cima. No Transformer, a atenção calcula dinamicamente quanto cada token deve influenciar os outros.

### Tokens

Os tokens são pedaços de um texto, são geralmente utilizados como unidade básica para usos diversos como a quantidade de contexto e preços da API. Você já deve ter ouvido por aí sobre os preços da Anthropic, eles cobram por milhão de tokens.

![Mostra a imagem de preço dos tokens](/images/posts/llms/tokens.png)

A ferramenta responsável por dividir o texto em tokens se chama tokenizer. Além disso, esses tokens servem como uma parte importante para o input de uma LLM. Depois que o texto vira tokens, cada um vira um ID, esse mapeamento de vocabulário varia de modelo pra modelo. Cada ID é passado em uma matriz de embeddings que gera um vetor de números, que contém uma representação numérica que foi aprendida. 

![Imagem representando a matriz de vetor de números](/images/posts/llms/PE3.png)
> Imagem tirada do post ["A Gentle Introduction to Positional Encoding in Transformer Models, Part 1"](https://machinelearningmastery.com/a-gentle-introduction-to-positional-encoding-in-transformer-models-part-1/)

É calculado também um vetor senoidal único para cada posição da sequência, usando várias ondas com frequências distintas.

{{< katex >}}

$$
PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$
> Fórmula tirada do paper "Attention Is All You Need".

Não são utilizados os índices, por exemplo índice 0 é a primeira e assim por diante. O motivo é que não escalariam bem pra sequências longas (os valores cresceriam sem limite) e não generalizariam pra tamanhos de sequência não vistos no treino. Esses dois vetores, o embedding (o quê) e o encoding posicional (onde), são então somados.

### Atenção

A atenção é uma projeção composta por três elementos: Q(Query), K(Key) e V(Value). Um exemplo sobre o que cada um representa:

```
Query   = o que está procurando.
Key     = como cada token se mostra para ser achado. 
Value   = a informação que aquele token carrega

Query: "quero algo relacionado a quem é raivoso".
Key de "cachorro": "animal / possível sujeito".
Value de "cachorro": informação representada sobre "cachorro".
```

Pegamos por exemplo a frase:
```text
O cachorro raivoso.
```

Dividindo em tokens ficaria:
```text
O (Q1, K1, V1)
cachorro (Q2, K2, V2)
raivoso (Q3, K3, V3)
```

Cada token terá seu Q, K e V. Será pego o valor de Q e comparado com a Key de cada um dos outros tokens da sequência: nesse caso, tanto cachorro quanto raivoso. A proximidade da Query com a Key determinará o quão influente será o V de cada um; vou definir esse termo como influência. Esse processo se repetirá na outra palavra, ficando assim:

```text
O (Q1, K1, V1 + V2 * influência + V3 * influência)
cachorro (Q2, K2, V2)
raivoso (Q3, K3, V3)
```

Esse processo se repetirá por cada token. Depois de calculados esses valores, acontece uma normalização chamada **Softmax** transformando em probabilidade. 

Por exemplo:
```
cachorro  3.5
raivoso   1.7
O         0.3
```

Depois do softmax:
```
cachorro  82.9%
raivoso   13.7%
O          3.4%
```

Isso tudo junto é uma explicação para a seguinte fórmula:

$$
\text{Attention}(Q,K,V)=\text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

$$
\underbrace{QK^T}_{\text{similaridade}} \rightarrow \underbrace{\frac{QK^T}{\sqrt{d_k}}}_{\text{escala}} \rightarrow \underbrace{\text{softmax}}_{\text{pesos de atenção}} \rightarrow \underbrace{\times V}_{\text{combinação das informações}}
$$

Essa parte aqui merece explicação:
$$
{\sqrt{d_k}}
$$

Esse trecho garante que os valores não cresçam demais, isso ajuda a evitar que o softmax deixe os valores concentrados demais em um token.

## Diferença de Softmax da Atenção para o Softmax da Saída

O primeiro eu já expliquei anteriormente, agora o da saída. Estamos agora na parte onde o LLM começa a escrever o texto, o softmax nesse estado é usado para prever qual token deve ser o próximo a ser escrito. Então é calculada a probabilidade da próxima palavra ser escrita, veja que disse probabilidade. Agora, a pergunta é: como você decide a palavra? Existem algumas formas: você pode escolher o que tem a maior probabilidade, que é chamado de `argmax`, ou deixar randomizado. O que é geralmente utilizado pelos grandes vendors é o randomizado, por isso seu input não gera sempre o mesmo texto de saída. Caso não tenha ficado claro, aqui vai uma analogia: é como jogar um dado viciado, ele tende a cair mais para um lado (o que tem a maior probabilidade), mas pode cair em outro lado também.

Com tudo isso, dá pra parecer que ficou bem mais complexo do que o modelo simples do começo do post, e ficou mesmo. Mas por baixo de Q, K, V, atenção e os dois softmax, o treinamento continua sendo os mesmos passos: Forward Pass, Loss, Backpropagation, Gradient Descent. O Transformer só empilha uma arquitetura bem mais sofisticada em cima disso antes de chegar numa saída.

## Referências

- https://www.cloudflare.com/pt-br/learning/ai/what-is-large-language-model/
- https://blog.nvidia.com.br/blog/o-que-e-um-modelo-transformer/
- https://www.ibm.com/br-pt/think/topics/large-language-models
- https://www.ibm.com/br-pt/think/topics/neural-networks
- https://www.redhat.com/en/blog/what-even-harness-ai
- https://huggingface.co/docs/transformers/main_classes/tokenizer
- https://www.ibm.com/think/topics/positional-encoding
- https://machinelearningmastery.com/a-gentle-introduction-to-positional-encoding-in-transformer-models-part-1/
- https://arxiv.org/abs/1706.03762
- https://www.3blue1brown.com/?topic=neural-networks
- https://www.ibm.com/br-pt/think/topics/deep-learning
- https://www.ibm.com/think/topics/gradient-descent
