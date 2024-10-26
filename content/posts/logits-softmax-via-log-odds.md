---
date: 2024-10-26
lastmod: 2024-10-26
showTableOfContents: false
tags: ["ml", "ai", "maths", "explainer", "orbit"]
title: "An intuitive understanding of logits and softmax via log-odds"
type: "post"
---
In modern ML/AI, logits and softmax are ubiquitous.
I provide an intuitive understanding of logits and softmax via log-odds, which most people are not aware of.

This post uses (an early version of) the [Orbit](https://withorbit.com/) system developed by Andy Matuschak.

## What are odds?

Odds are an alternative method of describing a probability distribution, where you specify the relative ratios of probabilties, instead of the probabilities themselves.

Example:
- Let $0.8, 0.1, 0.05, 0.05$ be the probabilities of the events $A, B, C, D$ respectively.
- This can be represented as the odds $16:2:1:1$.

The way to read this is that $A$ is $8$ times more likely than $B$ (because $16/2 = 8$).
Similarly, $B$ is $2$ times more likely than $C$ and $D$, and $C$ and $D$ are equally likely.

Because only the relative ratios matter, you can multiply all the odds by a constant to get a different representation of the same probability distribution.
E.g. $32:4:2:2$ and $800:100:50:50$ are valid representations of the example distribution.
In practice, you choose which ever representation is most convenient.

{{<orbit_review>}}
Suppose the odds for three events $A, B, C$ are $5:2:8$. How much more likely is $C$ than $B$?
$C$ is $4$ times more likely than $B$ because $8/2 = 4$.

Suppose the odds for $n$ possible outcomes are $o_1, o_2, \ldots, o_n$. How do you convert to probabilities?
You divide everything by the sum of all the odds.
{{</orbit_review>}}


## What are log odds?

To get from odds to log odds, you just take the logs!
For simplicity, we use base 2 in the example, but in practice you use the natural logarithm (base $e$).

Example:
- The probabilities of the events $A, B, C, D$ are $0.8, 0.1, 0.05, 0.05$ respectively.
- The odds are $16:2:1:1$.
- The log odds are $4:1:0:0$. (Because, for example, $2^4 = 16$ or $2^0 = 1$.)

The way to read log odds is that the *difference* between the log odds tells us the relative ratio of the probabilities. Explicitly:
- $A$ is $2^3 = 8$ times more likely than $B$. We get $3$ from $4 - 1$.
- $B$ is $2^1 = 2$ times more likely than $C$ and $D$. We get $1$ from $1 - 0$.

Remember that in practice, we use base $e$ instead of base $2$.

{{<orbit_review>}}
Suppose the log odds for three events $A, B, C$ are $5:1:8$. How much more likely is $A$ than $B$?
$A$ is $e^4$ (which is roughly 50) times more likely than $B$ because $5 - 1 = 4$.

Suppose the log odds for $n$ possible outcomes are $l_1, l_2, \ldots, l_n$. How do you convert to probabilities?
You first exponentiate everything to get odds, then divide by the sum of all the odds.
{{</orbit_review>}}


## Logits and softmax via log odds

- Big idea 1: Logits are just log odds.
- Big idea 2: Softmax is precisely the function that converts log odds to probabilities.

This means that the next time you see the outputs of a (classification) model, you can think of them as log odds.
In particular, you can interpret the numbers by remembering that the difference between the log odds tells you the relative ratio of the probabilities, e.g. a difference of $1$ means that the model is predicting one class as roughly $2.7$ times more likely than another.

An important detail to keep in mind is that in order for this interpretation of logits as log odds to be valid, the training of the model must be done in a way that the outputs are calibrated to be probabilities.
(This is big benefit of using cross entropy loss.)

An example where logits should *not* be interpreted as a probability distribution is in the attention mechanism of transformers.
Instead, it is better to think of the logits as (the log odds of) how much *weight* each token is putting on the other tokens.
There are no probabilities here and everything is deterministic.

## Benefits of using log odds

I know two main benefits. If you know others, please share!

- Any set of numbers can be log odds, whereas probabilities must be non-negative and sum to $1$.
This means that neural network does not need to learn these contraints during its training.

- Gradients, and hence the learning, is nicer in log space than in probability space.
In probability space, gradients are roughly equal to zero when the probability is close to zero.

{{<orbit_review>}}
What is the intuitive interpretation of logits given in this blogpost?
Logits are the log odds.

What is the intuitive interpretation of softmax given in this blogpost?
Softmax is the function that converts the log odds to probabilities.
{{</orbit_review>}}


## Exercises to check understanding

Note:
- For Q1, the aim is to answer using intuitive reasoning without having to do calculations / explicit algebra.
- I do not give answers.
It is a vital (and uncommon) skill to be able to check your own understanding without having to rely on a random internet stranger to tell you if you are right or wrong.

The exercises:
1. Suppose we have some log odds $1:1:1:3:4$.
    - What happens to the probabilities if I add $5$ to all the log odds (so we get $6:6:6:8:9$)?
    - What happens to the probabilities if I multiply all the log odds by $2$?
    - What happens to the probabilities if I multiply all the log odds by some huge number, e.g. $10^{10000}$?
    - What happens to the probabilities if I multiply all the log odds by some tiny number, e.g. $10^{-10000}$?
    - Use the above to explain why temperature 0 corresponds to the argmax function.
1. Show that the softmax function actually is the function that converts the log odds to probabilities.