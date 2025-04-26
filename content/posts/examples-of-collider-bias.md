---
date: 2021-02-21
lastmod: 2021-02-21
showTableOfContents: true
tags: ["data science", "causality", "tutorial"]
title: "Examples of collider bias"
type: "post"
description: "Collider bias can completely skew research findings. I describe some examples that highlight how this non-obvious bias arises."
---

## Introduction
In this post, I will describe four different examples of collider bias. In the past couple of months, it is probably the single biggest idea I have learnt and opened my eyes to new possibilities and subtleties in data. In particular, it reveals and explains why conditioning on some variable Z when you are trying to understand the effect of X on Y can make things worse; I used to think one ought to condition on as many variables as possible!

I will describe a few examples that illustrate what collider bias is, rather than giving a formal definition, because I admittedly do not yet know the formal definition of a collider!

The examples I describe are from [Causal Reasoning: The Mixtape](https://mixtape.scunning.com) and from [this interview of Peter Tenant](https://www.stitcher.com/show/the-turing-podcast/episode/being-an-epidemiologist-in-2020-76960769) on the Turing Podcast.

## Main intuition of collider bias
The main intuition I have is that a variable Z is a collider if by conditioning on Z you create a spurious correlation between two other variables A and B (which in turn muddles up our analysis of the causal effect of X on Y).

## Example 1. Movie stars.
Let us assume that to become a highly successful movie star, you must be either particularly talented or particularly good-looking (yes yes, looks are subjective, but you know what I mean). Let us also reasonably assume that in the whole population, there is no correlation between talent and looks.

With that set up, we can now say that if you condition on whether someone is a movie star, you create a negative correlation between talent and looks.

For a fuller explanation and some diagrams to visualise what is happening, see Section 3.1.6 from [Causal Inference: The Mixtape](https://mixtape.scunning.com/dag.html).

## Example 2. Elite students.
This is similar to the movie stars example, except this time we have the effect that by conditioning on whether somebody is at an elite institution or not will create a spurious negative correlation between talent and hardwork. A genius does not need to work hard to succeed whereas merely excellent individuals also need to work hard to get in.

## Example 3. Low birth weight paradox
This is a real example and it stumped the medical community for many years. The effect of smoking on infant mortality was being investigated. It was already known that smoking caused a higher risk of low-birth weight and that low birth weight increases the risk of infant mortality. We would like to know whether smoking has any other effects on infant mortality apart from causing low birth weight. So, you do what is natural and you condition on birthweight and now look at the effect of smoking on mortality. This is where the paradox comes in: it was found that smoking had a protective effect!

One's first instinct is there is a mistake in the data, but that is not the case. The explanation, as you might guess, is that it is a result of collider bias! Here are the relevant facts:
* There are various causes of birthweight. Smoking is one of them, but there are others.
* The other causes of low birthweight have a greater negative risk on infant mortality, separate to their effect on low birthweight.

As a challenge, combine all these facts to work out how conditioning on birthweight can create the finding that smoking has a protective effect.

...

...

...

Really, have a go. You learn more by actively engaging with an idea, than just reading somebody else's thoughts.

...

...

...

The explanation is as follows.
* When you condition on birthweight, you create a correlation between smoking and the other causes of birthweight. Specifically, a negative correlation.
* Hence, more smoking 'causes' less of the other causes of low birthweight, which causes a reduction in mortality.

If you did not follow that, there is a good chance my explanation is not ideal. I recommend you read through [Chapter 3 on Causal Diagrams](https://mixtape.scunning.com/dag.html), and then look at [Peter Tennant's tweet](https://twitter.com/pwgtennant/status/1030896340891586560?lang=en) containing the causal diagram for this situation.

## Example 4. Gender pay gap
When understanding the gender pay gap, it is obvious that one should condition on the job. There is a big difference between:
* Pay gap being a result of women being paid less than men for the same job
* Pay gap being a result of women tending to do lower paid jobs than men do

However, the thing that is not obvious is that conditioning on the job can create a spurious correlation. Let us make the following two assumptions:
* Women face discrimination in the job-hunting process
* There is a positive correlation between ability/talent and pay

Now, conditioning on jobs will create a spurious correlation between gender and ability - because women are discriminated against in the job-hunting process, women will tend to have a higher ability then men in the same job. Because of the second assumption, we would then observe that women are paid more than men, after conditioning on jobs!

To summarise, in this simplistic model women are facing discrimination but the standard approach of conditioning on jobs will make it look like it is in fact men who are being discriminated against.

For a fuller discussion and explanation (along with causal diagrams), see [Section 3.1.5 in Causal Inference The Mixtape](https://mixtape.scunning.com/dag.html).

## Conclusion
I hope these examples of collider bias were insightful. Like I said at the start, the main insight for me was that conditioning on a variable can sometimes create noise, and the reason is that if two variables A and B both cause Z, then conditioning on Z will create a spurious correlation between A and B.

A worrying thing about this is it reveals just how difficult answering causal questions is. As seen in the birthweight and gender examples above, you can observe the totally opposite effect as a result of collider bias. As far as I can tell, the only way to avoid collider bias is to have considered all the possible causes and effects, and to appropriately condition on the necessary variables. I guess I will just have to add the clause 'assuming all relevant causes have been taken into account' at the start of any claim I now read! 