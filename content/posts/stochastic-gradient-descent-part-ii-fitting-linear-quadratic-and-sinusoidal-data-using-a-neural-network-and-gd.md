---
date: 2020-09-11
lastmod: 2020-09-11
showTableOfContents: true
tags: ["data science", "neural network", "python"]
title: "Stochastic Gradient Descent, Part II, Fitting linear, quadratic and sinusoidal data using a neural network and GD"
type: "post"
description: "I continue my project to visualise and understand gradient descent. This time I try to fit a neural network to linear, quadratic and sinusoidal data."
---
## Other posts in series
{% for post in site.posts %}
{% if (post.title contains "Stochastic Gradient Descent, Part") and (post.title != page.title) %}
* [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endif %}
{% endfor %}


## Introduction
In the previous post, I showed my attempts at using gradient descent to fit linear, quadratic and sinusoidal data using (respectively) linear, quadratic and sinusoidal models. However, the universal approximation theorem says that the set of vanilla neural networks with one hidden layer can approximate any function to arbitrary precision. (An excellent and interactive sketch proof of this, where I first learnt about this theorem, is given in [Michael Nielsen's online book on neural networks](http://neuralnetworksanddeeplearning.com/chap4.html).) Therefore, it should be possible for a neural network to model the datasets I created in the first post, and it should be interesting to see the visualisations of the learning taking place.


## Linear data
I created some linear data `y = a*x + b + noise`, and then tried to fit a neural network to it.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_linearnn_1.mp4" type="video/mp4">
  </video>
</figure>

This is enchanting. I never thought I'd see something so delicate and graceful from crunching a whole bunch of numbers. I could watch this over and over again.

You will probably notice that the learning suddenly speeds up at about 17 seconds in the video. This is a result of me increasing the learning rate (from `1e-5` to `1e-4`) at a certain cut-off point. It is nice to be able to visually see the effect of changing the learning rate. Regarding the learning itself. it seems that the neural network struggles to fit the line well.

I next tried increasing the learning rate. This next video is an example when the learning rate was `1e-3` throughout.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_linearnn_3.mp4" type="video/mp4">
  </video>
</figure>

Changing the learning rate has improved the performance of the neural network considerably. The video is not as calm and satisfying to watch as the first one (though there is something comical about the jerky movements at the start), but it illustrates the value in choosing a good learning rate.

I next tried introducing a cutoff point where the learning rate increases from `1e-3` to `3e-3`. I have two examples of this.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_linearnn_5.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_linearnn_6.mp4" type="video/mp4">
  </video>
</figure>

In both of these videos, there is a moment in the middle where things go a bit wacky for a few iterations and then quickly settle back down. This wacky moment occurs precisely when the increase in the learning rate kicks in. My explanation for this is that we are seeing the algorithm jump away from one local minimum and moving towards another.

One thing I should have said is that in each of these videos, the training dataset remains the same, but the initialisation of the parameters of the neural network are different. So one other thing we are witnessing from these experiments is how different initialisation results in different models using gradient descent - i.e. there are many local minimums in the parameter space!



## Quadratic data
I created some quadratic data `y = a*x*x + b*x + c + noise` and tried to fit a neural network to it. Below are three examples. Note that as above, the dataset is staying the same each time, but the parameters are initialised differently each time and I play with the learning rates a bit, too.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_quadraticnn_1.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_quadraticnn_2.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_quadraticnn_4.mp4" type="video/mp4">
  </video>
</figure>

All these examples have similar overall behaviour. It is quick to get close-ish to the quadratic, and then the learning dramatically slows (but there is still learning going on throughout). Again we see how the different initialisations leads to different final models, showing how we are finding different local minimums.


## Sinusoidal data
I created some sinusoidal data `y = a*sin(b*x+ c) + d` and tried to fit a neural network to it. As before, same dataset but with different starting parameters.

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_1.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_2.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_3.mp4" type="video/mp4">
  </video>
</figure>


The first one shows some oscillating behaviour, similar to the first sinusoidal case in the first post of this series. Looks like the behaviour is not as unlikely as I previously thought. The second two produce mediocre results, at best.

Looking at the graphs, there seem to be 'too many wiggles', so I thought I'd try reducing the number of neurons in the hidden layer. The next three videos show what happens with eight neurons.



<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_4.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_5.mp4" type="video/mp4">
  </video>
</figure>

<figure class="video_container">
  <video controls="true" allowfullscreen="true">
    <source src="/images/sgd2_sinnn_6.mp4" type="video/mp4">
  </video>
</figure>

These perform less well than with twenty neurons. The final video is interesting for its crazy behaviour at the start - the function seems to spin around and do all kinds of weird stuff before settling down. I do not know what to make of that.

There is lots more experimentation that can be done, but I feel this is a good place to stop and move on to other things to investigate.


## Conclusions
There are various things I learnt doing this.

* There is a lot to be learnt by playing around with the ideas. Though (I think) I understand the theory of gradient descent and of vanilla neural networks, it is evident that the whole is greater than the sum of its parts. I under-estimated what I could have learnt by experimenting.

* It seems neural networks are only useful for interpolation - i.e. making predictions for data similar to the training data. They are hopeless at extrapolating: I think they necessarily have horizontal asymptotes in both x-directions.

* The initialisation of parameters matters a lot for neural networks.
    * My very first settings were useless, and were seemingly only capable of modelling a sigmoid function. For a while, I thought my code was buggy, but after some experimentation, it turned out the initial parameter settings were inappropriate.
    * Different initialisations resulted in vastly different final models. Presumably this is also the case with more complex tasks, but maybe it does not matter when doing a classification on real data.
    
* The range of values in the data impacts the choice of parameters. As I mentioned in the first post, I have read that normalising is important. The fact that the parameter settings are sensitive to the range of values in the dataset could be a big reason why normalising is important.

* Manually creating a neural network (even with only 1 layer) is a bit of faff; I should learn how to create one using the pytorch library.

* You can create some funky visuals with neural networks! Somebody more creative than I am could add some music to each of these to produce some interesting art. These animations may even be a weird alternative to [Rorschach inkblot tests](https://en.wikipedia.org/wiki/Rorschach_test): 'How does this video make you feel? What do you see in this video?'



## The code
The code for this project is in this [GitHub repository](https://github.com/Lovkush-A/pytorch_sgd). I encourage you to play around and see what you can learn. If there is anything you do not understand in the code, please ask. 