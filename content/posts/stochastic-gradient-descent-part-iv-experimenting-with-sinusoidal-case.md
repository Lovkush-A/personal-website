---
date: 2020-10-01
lastmod: 2020-10-01
showTableOfContents: true
tags: ["data science", "python"]
title: "Stochastic Gradient Descent, Part IV, Experimenting with sinusoidal case"
type: "post"
description: "I end this series by describing some experiments I did with the sinusoidal case, before I realised that the learning rate was too big. Spoiler alert: turns out my first instincts from Part I were correct all along..."
---
## Other posts in series
{% for post in site.posts %}
{% if (post.title contains "Stochastic Gradient Descent, Part") and (post.title != page.title) %}
* [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endif %}
{% endfor %}


## Introduction
I will recap my investigations into fitting sinusoidal data using sinusoidal models with SGD.
* In Part I, I described my attempt at using GD and my belief that the failure to fit was due to the model getting stuck in a local minimum where the amplitude is small. I hoped that SGD would fix this problem.
* I tried using SGD but it did not help (described in Part III).
* I then tried using regularisation to solve the amplitude issue (described below).
* I had the idea of investigating the loss function in detail. However, I decided it would be better to separate off this detailed investigation into sinusoidal models into a separate post (this one), and have a post discussing only the stochasticity (Part III).
* While writing up Part III, for the sake of completeness, I investigated the learning rate. This solved the issue!
* However, even though I solved the issues, I thought it would still be worthwhile to write-up my experiments with regularisation and also to carry out the investigation into the loss function. So here we are!


## Regularisation
Based on the examples I had tried, the amplitude would always tend to zero. Hence, I thought it would be worth adding a regularisation term that punishes having small amplitudes.

The loss function was `loss = mse(y_est, y)`. After the regularisation, it became `loss = mse(y_est, y) - parameters_est[0]`.  Why did I choose this regularisation?

* I believed that the amplitude would naturally tend to small values. Thus, I want to punish small values and encourage large values.
* Smaller losses should correspond to better models. Therefore, the larger the amplitude, the smaller the loss should be.
* Subtracting the amplitude from the loss achieves this. (Note that the first element of `parameters_est` is the amplitude).
* By differentiating, this regularisation causes the amplitude to increase by a constant amount each step, so there is a constant upward pressure on the amplitude.

Below is the first result of introducing this regularisation.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin1.mp4" type="video/mp4">
  </video>
</figure>

As you can see, there are moments where the model gets close to the data. This got my hopes up, and made me feel like I was onto something.

I tried various other things to see if I could make it better. I tried changing the weight of the regularisation term. I tried adding other regularisation terms (because in the experiments, it looked like there was now a tendency for the frequency to keep increasing). I can't remember if I tried other things or not. Suffice it to say, I made no progress.

Below is an animation of an experiment which involved changing the weight of the regularisation term. I include it only because I thought it was particularly funky and visually interesting.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin2.mp4" type="video/mp4">
  </video>
</figure>


## Visualising the loss function
After failing to get regularisation to work, I decided I should try to visualise the loss function, and find out exactly where the local minima were, and hopefully better understand why things were not working.

The process I followed was:
* Create data to be fit. That was just `y = sin(x)`
* Create generic sinusoidal models `y_est = a*sin(b*x + c) + d`
* Vary the parameters `a,b,c,d` and calculate the loss, `mse(y, y_est)`
* Plot graphs to visualise the loss function

To begin, I set `c` and `d` to `0` and varied `a` and `b`. `a` is the amplitude and `b` is the frequency (multiplied by `2*pi`) or the coefficient of `x`. The reason for fixing `c` and `d` is that it was the amplitude and the frequency which were giving the most trouble.


The first animation below shows a sequence of charts. Each individual chart shows how the loss varies with frequency, and from chart to chart the amplitude is changing.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_loss_vs_freq.mp4" type="video/mp4">
  </video>
</figure>

As can be seen from this, there are many local minima, so the model might get stuck in the wrong one. Eye-balling the chart, if the initial frequency is below 0.5 or above 1.7, then gradient descent will push the frequency away from the optimal value of 1. It is now clear why there should be a tendency for the frequency to increase, as we saw in the SGD examples in Part III.

The next animation is the opposite. For each individual chart, we see how the loss varies with amplitude, and from chart to chart we are modifying the frequency.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_loss_vs_amp.mp4" type="video/mp4">
  </video>
</figure>

Fantastic! This I feel like I can understand. For the majority of frequencies, the optimal value for amplitude is zero and the amplitude will just slide its way to that value. Only for a narrow range of frequencies is the optimal value of the amplitude non-zero.

To summarise, based on these two animations, here is what I would predict:
* Regardless of amplitude, there is a narrow band of frequencies which would result in SGD finding the global minimum. Otherwise, you will get stuck in some other local minimum.
* For 'small' and 'large' frequencies, the amplitude will want to decay to zero. For a certain range of frequncies, the amplitude will tend towards a sensible value.

As I am writing this up and thinking things through, I am starting to wonder about my conclusion in Part III about the sinusoidal model. In Part III, I concluded that the issue all along was having an inappropriate learning rate, but the two animations above suggest there is more to it. Did I just get lucky and stumble upon starting parameters which fit the criteria I described above, and hence that is why I got the sinusoidal model to fit?  There's only one way to find out, which is to do more experimentation!

## Investigating parameter initialisation
The steps for the investigation are as follows:
* Create data to be fit and generic model, as above.
* Initialise the estimated parameters: `a=1, b=?, c=0, d=0`. We will be varying the value of initial value of `b`
* Do SGD and visualise the learning

I start by trying a large value for the frequency, 5.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f5.mp4" type="video/mp4">
  </video>
</figure>

So, as predicted, the frequency gets stuck in some sub-optimal value and the amplitude tends to zero. It looks like I did just get lucky in Part III.

Frequency of 2 and 1.5 is similar:

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f2.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f15.mp4" type="video/mp4">
  </video>
</figure>


Frequency of 1.2:

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f12.mp4" type="video/mp4">
  </video>
</figure>

We get the model converging to the data! Though this is to be expected, it is still satisfying to see it actually work. With a bit of manual experimentation, the cut-off between these two behaviours is roughly 1.46.

How about lower frequencies? A frequency of 0.6 converges to the correct model:

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f06.mp4" type="video/mp4">
  </video>
</figure>

And a frequency of 0.5 converges to a different minima:

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd4_sin_f05.mp4" type="video/mp4">
  </video>
</figure>

Again, this is consistent with the frequncy vs loss charts above, where you can see there are local minima to the left of the global minimum.


## Conclusion
This has been a bit of a topsy-turvy learning experience. I am still surprised at how much I learnt from this basic example. And having struggled with this simple example, I better appreciate how impressive it is to get complicated neural networks to learn. 