---
title: "How does an LLM actually work?"
date: 2026-08-15
summary: "Going deep on how AI actually works."
tags: ["AI", "Work"]
---

It's undeniable that the launch of ChatGPT, in November 2022, had a huge impact, and big questions started popping up because of it. Then came the image-generating models, raising even more ethical questions about AI. Finally, more recently, with LLMs getting better, it became possible to actually use them for programming. My goal with this series of posts is to build deep dives into several topics, for people who know close to nothing all the way up to current use cases. In this first post of the series, I'm going to explain how LLMs work.

## About LLMs

LLMs (Large Language Models) are models trained on a massive amount of data in huge data centers. The quality of the model depends on the quality of the data used to train it.

![Shows the image of xAI's Memphis data center](/images/posts/llms/data-center.png)
> xAI's data center in Memphis. Photo: Steve Jones/Flight by Southwings for the Southern Environmental Law Center

Models can recognize and generate text. There are different architectures for these models, and the type of network I'll cover here, the one big models like ChatGPT and Claude use, is called Transformer. What changed from earlier models is that now they can process the tokens of a sequence in parallel and decide which ones should influence the others, through the attention mechanism. The architecture was presented in 2017 by Google researchers in the paper [“Attention Is All You Need”](https://arxiv.org/abs/1706.03762), which had a massive impact on the development of the AI field.

## Deep Learning

> "Deep learning is a subset of machine learning driven by multilayered neural networks whose design is inspired by the structure of the human brain. Deep learning models power most state-of-the-art artificial intelligence (AI) today, from computer vision and generative AI to self-driving cars and robotics."
> [IBM's explanation of the term](https://www.ibm.com/think/topics/deep-learning)

### Neural Networks

Neural networks, as the name suggests, are **inspired** by the brain. Pay attention to "inspired." The network is split into nodes that connect to each other, and those nodes are organized into layers.
- Input layer: receives the data.
- Hidden layers: process the information.
- Output layer: gives the answer.

The "neuron" holds a value inside called activation. And every connection into a neuron (except the ones on the first layer) has a weight, which gets adjusted during training. Those weights are essential to make sure the model lands on the correct answer.

{{< mermaid >}}
flowchart LR
    subgraph L1["Initial layer"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Next layer"]
        H1(("H₁<br/>a = 0.735"))
    end

    N1 -->|"w₁ = 0.50"| H1
    N2 -->|"w₂ = -0.30"| H1
    N3 -->|"w₃ = 0.80"| H1
    N4 -->|"w₄ = 0.10"| H1
{{< /mermaid >}}

When you actually use the model, whether it's ChatGPT or Claude, the weights are already fixed.

### Activation function

If you noticed, every activation number sits between 0 and 1. That happens because of the activation function used; for every neuron, you can compute its activation value. The one used here is Sigmoid.

$$ 
 \sigma(x) = \frac{1}{1 + e^{-x}}
$$

Every neuron, as you saw before, has other neurons pointing into it, each with its own value and weight. We can put all that together and build the formula to find the activation value.

$$
a_{\text{out}}=\sigma\left(\sum_{i=1}^{n} a_i w_i+b\right)=\frac{1}{1+e^{-\left(\sum_{i=1}^{n} a_i w_i+b\right)}}
$$

Calculating the value for the neuron in the mermaid example above:

$$
a_{H_1}=\sigma\left(0.8\cdot0.5+0.4\cdot(-0.3)+0.9\cdot0.8+0.2\cdot0.1\right)\approx0.735
$$

There are other, more complex methods that don't stick to the 0-1 range. I won't get into them here. If you're curious, that's on you to go look up. This activation function isn't used by state-of-the-art models; it's here just to help you understand part of what an LLM does under the hood.

Inside the activation function you can add a `bias`, which decides when the sum of the weighted activations should actually matter. In other words, you're adjusting the starting point.

$$
a_{\text{out}}=\sigma\left(\sum_{i=1}^{n}a_iw_i+b\right)
$$

### Gradient Descent

Ok, we've reached a cool point. We can already build our little Frankenstein and make some level of decision. Except there's a problem: manually adjusting every single one of these weights by hand just isn't feasible. My example used only **one**, now go plug in values for this one below and do the math.

{{<mermaid>}}
graph LR
    subgraph Entrada["Input Layer"]
        I1((x1))
        I2((x2))
        I3((x3))
    end

    subgraph Oculta1["Hidden Layer 1"]
        H1((h1))
        H2((h2))
        H3((h3))
        H4((h4))
    end

    subgraph Oculta2["Hidden Layer 2"]
        H5((h5))
        H6((h6))
        H7((h7))
    end

    subgraph Saida["Output Layer"]
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

See how that doesn't make sense. We'd need a simpler way to do this. I tell it what the expected answer was and it adjusts itself. To pull that off, I first need to teach the mechanism that adjusts the weights.

Let's recap in simple terms what a gradient means: it shows in which direction the function grows fastest. What we want is to minimize the error, so we need to go the opposite direction.

$$w_{new} = w_{current} - \eta \cdot gradient$$

Where:
- $w$: is the neural network's weight
- $\eta$: the learning rate

The learning rate value has to be chosen carefully: values that are too high can overshoot the ideal point and oscillate wildly, values too low can make it take forever to get there.

### Backpropagation

In the formula above there's the term `gradient`: the method used to calculate that gradient is backpropagation.
- Backpropagation = calculates the gradients, the slope of each weight.
- Gradient descent = uses those gradients to update the weights.

$$\dfrac{\partial L}{\partial w}$$

The partial derivative of the error function $L$ with respect to weight $w$. There are different methods for calculating the Loss function; I'll go with **Binary Cross-Entropy**.

### Example

Let's now put everything together. The training process works like this:

```text
Input → Forward Pass → Backpropagation → Gradient Descent → New Weights
```

This happens over and over, thousands or billions of times in large models. To make it clear what the Forward Pass is: it's testing by plugging in input values and observing the output result.

Let's use the network we saw earlier.
{{< mermaid >}}
flowchart LR
    subgraph L1["Initial layer"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Next layer"]
        H1(("H₁<br/>a = 0.735"))
    end

    N1 -->|"w₁ = 0.50"| H1
    N2 -->|"w₂ = -0.30"| H1
    N3 -->|"w₃ = 0.80"| H1
    N4 -->|"w₄ = 0.10"| H1
{{< /mermaid >}}

$$
a_1=0.8,\quad a_2=0.4,\quad a_3=0.9,\quad a_4=0.2
$$

The weights are:
$$
w_1=0.5,\quad w_2=-0.3,\quad w_3=0.8,\quad w_4=0.1
$$

And we'll define the correct answer for the output as:
$$a_{\text{out}}=1$$

#### Forward Pass

We already did this earlier, and the calculation gave us:
$$
a_{\text{out}}=\sigma\left(0.8\cdot0.5+0.4\cdot(-0.3)+0.9\cdot0.8+0.2\cdot0.1\right)\approx0.735
$$

#### Backpropagation

With this value we'll compute the partial derivative.
$$
\frac{\partial L}{\partial w_1}
$$

For **sigmoid and binary cross-entropy**, a very convenient simplification happens:

$$
\frac{\partial L}{\partial z}=\hat y-y
$$

Where:
- $\hat y$: the value we got.
- $y$: the value we wanted.

So:

$$
\frac{\partial L}{\partial z}=0.735-1
$$

$$
\frac{\partial L}{\partial z}=-0.265
$$

Now:

$$
z=a_1w_1+a_2w_2+\cdots
$$

therefore:

$$
\frac{\partial z}{\partial w_1}=a_1
$$

Since:

$$
a_1=0.8
$$

we get:

$$
\frac{\partial L}{\partial w_1} = (-0.265)(0.8)
$$

Which gives us:

$$
\frac{\partial L}{\partial w_1}\approx-0.212
$$

#### Gradient Descent

I'll pick $\eta=0.1$:
$$
w_1^{new} = 0.5-(0.1)(-0.212)
$$

$$
w_1^{new}\approx0.5212
$$

#### Forward again

Now using:

$$
w_1=0.5212
$$

we get:

$$
z=(0.8)(0.5212)+(0.4)(-0.3)+(0.9)(0.8)+(0.2)(0.1)
$$

$$
z\approx1.037
$$

Applying sigmoid:

$$
\hat y=\sigma(1.037)
$$

$$
\hat y\approx0.738
$$

Before:

$$
\hat y=0.735
$$

After:

$$
\hat y=0.738
$$

Since our target was:

$$
y=1
$$

the prediction moved in the right direction.

And the loss dropped too:

$$
0.308\rightarrow0.303
$$

So, **a single update already made the model slightly better in this example**. Keep in mind, this was on just **one** neuron. If we did this for every other weight, we'd end up with something like this:

{{< mermaid >}}
flowchart LR

    subgraph L1["Initial layer"]
        N1(("N₁<br/>a₁ = 0.8"))
        N2(("N₂<br/>a₂ = 0.4"))
        N3(("N₃<br/>a₃ = 0.9"))
        N4(("N₄<br/>a₄ = 0.2"))
    end

    subgraph L2["Next layer"]
        H1(("H₁<br/>z = 1.064<br/>a = 0.743"))
    end

    N1 -->|"w₁ = 0.5212"| H1
    N2 -->|"w₂ = -0.2894"| H1
    N3 -->|"w₃ = 0.8239"| H1
    N4 -->|"w₄ = 0.1053"| H1
{{</ mermaid >}}

## Where Models Stand Today

The model explained above is a foundation; in today's models, some things have changed and gotten a lot more sophisticated. I'll talk about the architecture most used today, the Transformer, used heavily in the big models. It's the architecture from the paper I mentioned earlier. In the Transformer, attention dynamically calculates how much each token should influence the others.

### Tokens

Tokens are chunks of text, generally used as the basic unit for things like context size and API pricing. You've probably heard about Anthropic's pricing: they charge per million tokens.

![Shows the image of token pricing](/images/posts/llms/tokens.png)

The tool responsible for splitting text into tokens is called a tokenizer. Beyond that, these tokens are an important part of an LLM's input. Once text becomes tokens, each one becomes an ID, and this vocabulary mapping varies from model to model. Each ID gets passed through an embedding matrix, which produces a vector of numbers, a numerical representation the model learned.

![Image representing the numerical vector matrix](/images/posts/llms/PE3.png)
> Image taken from the post ["A Gentle Introduction to Positional Encoding in Transformer Models, Part 1"](https://machinelearningmastery.com/a-gentle-introduction-to-positional-encoding-in-transformer-models-part-1/)

A unique sinusoidal vector is also calculated for each position in the sequence, using several waves with different frequencies.

{{< katex >}}

$$
PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$
> Formula taken from the paper "Attention Is All You Need."

Raw indices, like index 0 for the first position and so on, aren't used. The reason is that they wouldn't scale well for long sequences (the values would grow without bound) and wouldn't generalize to sequence lengths not seen during training. These two vectors, the embedding (the what) and the positional encoding (the where), are then added together.

### Attention

Attention is a projection made up of three elements: Q (Query), K (Key), and V (Value). An example of what each one represents:

```
Query   = what it's looking for.
Key     = how each token presents itself to be found.
Value   = the information that token carries

Query: "looking for something related to who is angry."
Key for "dog": "animal / possible subject".
Value for "dog": the information represented about "dog".
```

Take this sentence, for example:
```text
The angry dog.
```

Split into tokens, it becomes:
```text
The (Q1, K1, V1)
angry (Q2, K2, V2)
dog (Q3, K3, V3)
```

Every token has its own Q, K, and V. The Q value gets compared against the Key of every other token in the sequence (in this case, both "angry" and "dog"), and how close the Query is to each Key determines how influential that token's V will be; I'll call this "influence." This process repeats for the other words too, ending up like this:

```text
The (Q1, K1, V1 + V2 * influence + V3 * influence)
angry (Q2, K2, V2)
dog (Q3, K3, V3)
```

This process repeats for every token. Once these values are calculated, a normalization step called **Softmax** turns them into probabilities.

For example:
```
dog     3.5
angry   1.7
The     0.3
```

After softmax:
```
dog     82.9%
angry   13.7%
The      3.4%
```

All of this together is what the following formula represents:

$$
\text{Attention}(Q,K,V)=\text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

$$
\underbrace{QK^T}_{\text{similarity}} \rightarrow \underbrace{\frac{QK^T}{\sqrt{d_k}}}_{\text{scaling}} \rightarrow \underbrace{\text{softmax}}_{\text{attention weights}} \rightarrow \underbrace{\times V}_{\text{combining the information}}
$$

This part deserves an explanation:
$$
{\sqrt{d_k}}
$$

This term keeps the values from growing too large, which stops softmax from piling all its weight onto a single token.

## Attention Softmax vs. Output Softmax

I already explained the first one above; now let's talk about the output one. We're now at the part where the LLM starts writing text, and softmax at this stage is used to predict which token should come next. So it computes the probability of the next word. Notice I said probability. Now, the question is: how do you pick the word? There are a few ways: you can go with the highest probability, called `argmax`, or leave it randomized. What big vendors generally use is randomized sampling, which is why your input doesn't always produce the same output text. In case that wasn't clear, here's an analogy: it's like rolling a loaded die: it tends to land on one side more often (the highest-probability one), but it can still land on another.

With all of this, it probably looks way more complex than the simple model from the start of the post, and it is. But underneath Q, K, V, attention, and the two softmax steps, training still comes down to the same steps: Forward Pass, Loss, Backpropagation, Gradient Descent. The Transformer just stacks a much more sophisticated architecture on top of that before it produces an output.

## References

- https://www.cloudflare.com/learning/ai/what-is-large-language-model/
- https://blogs.nvidia.com/blog/what-is-a-transformer-model/
- https://www.ibm.com/think/topics/large-language-models
- https://www.ibm.com/think/topics/neural-networks
- https://www.redhat.com/en/blog/what-even-harness-ai
- https://huggingface.co/docs/transformers/main_classes/tokenizer
- https://www.ibm.com/think/topics/positional-encoding
- https://machinelearningmastery.com/a-gentle-introduction-to-positional-encoding-in-transformer-models-part-1/
- https://arxiv.org/abs/1706.03762
- https://www.3blue1brown.com/?topic=neural-networks
- https://www.ibm.com/think/topics/deep-learning
- https://www.ibm.com/think/topics/gradient-descent
