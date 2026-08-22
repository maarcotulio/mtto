---
title: "AI Post-Training"
date: 2026-08-22
summary: "Explaining what happens after pre-training: SFT, Reward Model, and Reinforcement Learning, the steps that turn the model into an aligned assistant."
tags: ["AI", "Work"]
---

In the previous post I explained pre-training, the most time-consuming and expensive part of the whole process. I talked about the Transformer, the architecture used by today's models, but left out the other important part of building an agent. With what we've covered so far, we have a model that's very good at completing text, but that still doesn't know how to generate a well-formatted response with any kind of personality.

## Supervised Fine-Tuning (SFT)

After the model has already been trained on absurd amounts of data across different topics, we now need to teach it to produce a more satisfying response in a conversation. So in this first stage, it's fed a bunch of example conversations: user questions and what the response should look like.

For example:
```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful, concise technical assistant."
    },
    {
      "role": "user",
      "content": "How can I check open ports on Linux using the terminal?"
    },
    {
      "role": "assistant",
      "content": "You can use the `ss` or `netstat` command with superuser privileges:\n\n* **With `ss` (recommended):**\n  ```bash\n  sudo ss -tulpn\n  ```\n* **With `netstat`:**\n  ```bash\n  sudo netstat -tulpn\n  ```\n\n**What the flags mean:**\n* `-t`: TCP connections\n* `-u`: UDP connections\n* `-l`: Listening ports only\n* `-p`: Shows the responsible process/PID\n* `-n`: Shows port numbers instead of service names"
    }
  ]
}
```

The method used for this part of training follows a line pretty similar to what we learned in the previous post.

> So why is this process split up?

Each of these training stages solves a different objective. If you tried to do everything at once, you'd likely lose quality somewhere along the way. Pre-training builds the foundation: it needs huge amounts of data, and that's where the AI picks up its knowledge. SFT, on the other hand, needs higher quality data, and it's what makes the model generate text with a more appropriate format and style.

## Reward Model

In this stage, a separate neural network is trained to act as a critic of the responses, the Reward Model (RM). It doesn't generate text; it takes a prompt and a response and returns a number saying how good that response is.

To train this critic, you take the model already fine-tuned by SFT and generate two or more different responses for the same prompt. A human then compares these responses and says which one they prefer: no need to give an exact score, just point out "this one's better than that one." With enough of these comparison pairs, the Reward Model learns to predict a score that tries to mirror that human judgment.

## Reinforcement Learning (RL)

{{< katex >}}

Now comes the part where the model (here called the `policy`) generates responses to a prompt, the Reward Model scores that response, and that score becomes the signal used to update the model's weights, with no human in the loop needed at this point anymore. The method OpenAI used in the InstructGPT paper (the same pipeline used to train ChatGPT) to do this update is called **PPO** (Proximal Policy Optimization).

There's a problem, though: if the objective is simply to "maximize the Reward Model's score," the model can find a way to fool the critic, generating responses that score high but that are actually bad or don't even make sense. This is called **reward hacking**. To prevent this, the actual training objective also penalizes how far the new policy has drifted from the original SFT model:

$$
\max_{\pi} \; \mathbb{E}_{x,\, y \sim \pi}\big[r(x, y)\big] \;-\; \beta \, D_{KL}\big(\pi \,\|\, \pi_{ref}\big)
$$

Where:
- $r(x, y)$: the score the Reward Model gives to response $y$ for prompt $x$.
- $\pi$: the current policy, the model being trained.
- $\pi_{ref}$: the SFT model, used as reference.
- $D_{KL}$: the KL divergence, a measure of how different $\pi$ has become from $\pi_{ref}$.
- $\beta$: controls how much weight this distance gets against the reward.

In other words: the model is free to improve by chasing higher scores, but it gets "pulled back" whenever it tries to drift too far from the behavior it already learned during SFT.

![Diagram of the three RLHF stages: Supervised Fine-Tuning, Reward Model training, and policy optimization with PPO](/images/posts/postreino/image.png)
> Image taken from [AWS](https://aws.amazon.com/what-is/reinforcement-learning-from-human-feedback/)

These days it's also common to see models tuned with **DPO** (Direct Preference Optimization), a more recent and simpler alternative: it gets the same effect of aligning the model to human preferences without needing to train a separate Reward Model or run PPO, using the preferred/rejected response pairs directly in the loss function.

In the end, SFT, the Reward Model, and RL (whether with PPO or DPO) are the three pieces that take that model, which only knew how to complete text, and turn it into an assistant that answers in a useful way, aligned with what we expect. There's still a part missing to make it a real agent, capable of using tools and acting on its own, but that's for the next post.

## References

- https://arxiv.org/abs/2203.02155 (InstructGPT: originates the SFT → Reward Model → PPO pipeline)
- https://arxiv.org/abs/1707.06347 (PPO)
- https://arxiv.org/abs/2305.18290 (DPO)
- https://huggingface.co/blog/rlhf
- https://aws.amazon.com/what-is/reinforcement-learning-from-human-feedback/
- https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf
- https://aws.amazon.com/what-is/reinforcement-learning/
- https://arxiv.org/pdf/2204.05862
