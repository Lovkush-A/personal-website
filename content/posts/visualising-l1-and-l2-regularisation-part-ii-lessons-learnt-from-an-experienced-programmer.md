---
date: 2020-10-18
lastmod: 2020-10-18
showTableOfContents: true
tags: ["data science", "python"]
title: "Visualising L1 and L2 regularisation, Part II, Lessons learnt from an experienced programmer"
type: "post"
description: "The process I used to make the animations was inefficient and not programmatic. I could not work out how to adapt the matplotlib animation tools to my situation so I asked for help. Here I describe what I learnt from the help that I received."
---
## Other posts in series
{% for post in site.posts %}
{% if (post.title contains "L1 and L2") and (post.title != page.title) %}
* [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
{% endif %}
{% endfor %}

## Introduction
In the first post in this series, I produced various animations to help visualise L1 vs L2 regularisation. However, the way I produced those animations was not 100% programmatic. This is what I did:
* Have a for loop which produces each chart then saves it as a png file.
* Manually use an online gif tool to combine the png files.

However, I knew it was possible to create a video programmatically in matplotlib, as I had done it before to [visualise gradient descent]({% post_url 2020-09-10-sgd1 %}). But I could not work out how to adapt the functions to this case.  Therefore, I decided to ask for help in the Faculty Slack channel. I got two helpful responses.

* One by Will Fawcett which told me about the command line tool `convert` that can create the gifs
* Second by Tom Begley who recommended using `FFMpegWriter`. Furthermore, he actually created a pull request in which he adapted my code to illustrate how to use it!

This was the first time I had asked for help in the general Slack channel, so I was taken aback by the help that was provided.

Anyway, in the pull request, in addition to illustrating how to use `FFMpegWriter`, there were various other little things that were changed and things I could learn from. Therefore, I am writing this blogpost to maximise how much I learn from the experience.

## Lessons learnt
### Automatic code formatting for notebooks
A noticeable change in the pull request was that many of the changes concerned code formatting. Below is an example.

Before:
```python
def create_cost_fn(centre_x, centre_y, reg=None,  reg_const=0):
    """
    returns a convex cost function
    """
    if reg is None:
        def cost(x,y):
            return (x - centre_x)**2 + (y - centre_y)**2
    elif reg == 'l1':
        def cost(x,y):
            return (x - centre_x)**2 + (y - centre_y)**2 + reg_const*(abs(x) + abs(y))
    elif reg == 'l2':
        def cost(x,y):
            return (x - centre_x)**2 + (y - centre_y)**2 + reg_const*(x**2 + y**2)
    elif reg == 'max':
        def cost(x,y):
            return (x - centre_x)**2 + (y - centre_y)**2 + reg_const*(x**10 + y**10)**0.1

    return cost
```

After:
```python
def create_cost_fn(centre_x, centre_y, reg=None, reg_const=0):
    """
    returns a convex cost function
    """
    if reg is None:

        def cost(x, y):
            return (x - centre_x) ** 2 + (y - centre_y) ** 2

    elif reg == "l1":

        def cost(x, y):
            return (
                (x - centre_x) ** 2
                + (y - centre_y) ** 2
                + reg_const * (abs(x) + abs(y))
            )

    elif reg == "l2":

        def cost(x, y):
            return (
                (x - centre_x) ** 2
                + (y - centre_y) ** 2
                + reg_const * (x ** 2 + y ** 2)
            )

    elif reg == "max":

        def cost(x, y):
            return (
                (x - centre_x) ** 2
                + (y - centre_y) ** 2
                + reg_const * (x ** 10 + y ** 10) ** 0.1
            )

    return cost
```

I knew there was no way Tom did this manually so surmised he used an automatic code formatter. A quick Google search revealed various options and I will be sure to make use of these in the future.

### Decorators
I already knew about decorators, but somehow never thought of making use of them.

Before:
```python
def create_y_l1_coordinate(x, r):
...
create_y_l1_coordinates = np.vectorize(create_y_l1_coordinate)
```

After:
```python
@np.vectorize
def create_y_l1_coordinate(x, r):
...
```

### Garbage collection
Garbage collection is something I have briefly read about, but have not fully understood. However, a certain aspect of Tom's example gave me some insight. It is best explained by showing the example. Tom created the `update` function within an `initialise` function:

```python
def initialise(angles, ball_radius=4, norm="l1"):
    centres_x = 5 * np.cos(angles)
    centres_y = 5 * np.sin(angles)

    f, ax = plt.subplots(figsize=(10, 10))

    x_ball, y_ball = create_ball_boundary(4, "l1")
    # comma is needed to pull poly out of list length 1
    ax.fill(x_ball, y_ball, "b", alpha=0.2)

    (marker,) = ax.plot(
        0,
        0,
        marker="o",
        color="red",
        label="Minimum value of cost within shaded region",
    )
    ax.legend()

    def update(i):
        cost = create_cost_fn(centres_x[i], centres_y[i])
        (
            cost_inputs_x,
            cost_inputs_y,
            cost_outputs,
        ) = create_cost_inputs_outputs(cost)
        cont = ax.contour(
            cost_inputs_x,
            cost_inputs_y,
            cost_outputs,
            levels=np.arange(0, 51, 2.5),
        )

        x_min, y_min = minimise_cost(cost, x_ball, y_ball)
        marker.set_xdata(x_min)
        marker.set_ydata(y_min)
        return cont

    return f, ax, update
```

Before seeing this example, I would have said that `centres_x`, `centres_y` and `marker` would be 'garbage collected' and the variables would not be preserved. However, the fact the program works means that those variables must be preserved. Looking at the code, the only way this is possible is because they are used in the `update` function (and the fact that `update` is returned by `initialise`).

Though I cannot say I fully understand what is going on, I do have a better appreciation for garbage collection in python, and how there is a counter that tracks the number of things that are somehow using/referring to a python object.

Lastly, this example highlights some things I have read about bad programming practice. If something went wrong, the update function would be hard to debug, as it is making to changes to variables that are not parameters to the function.  I do not mind it in this example, as Tom was trying to illustrate how to use FFMpegWriter and it is not worth his time to use optimal programming practice in some little learning project. Furthermore, it has indirectly increased the amount I have learnt from the experience!


### How to use `FFMpegWriter`
Of course, the main thing I learnt from Tom's help was how to use `FFMpegWriter` and adjust for the different nature of these plots compared to other animation examples I had seen/created previously.

```python
f, ax, update = initialise(angles)
writer = FFMpegWriter(fps=10)

with writer.saving(f, "l1-cost.mp4", 100):
    for i in range(len(angles)):
        cont = update(i)
        writer.grab_frame()
        remove_contours(cont)
```

### How how I might have found the solution myself
I asked Tom: 'How would I have found out about the syntax to remove the contours, given that standard animation examples use 'updates' (e.g. you use updates for the red marker).  What would I have googled for, or what documentation would I have read?'

Their response: 'Yeah that was the nastiest bit here. Normally I look at the return value of whatever plotting function I used and try to tab-complete `.set_` to see what's available. In the case of the contour there was nothing obvious so I googled something like "update matplotlib contour animation" and got [this stackoverflow answer](https://stackoverflow.com/questions/23250004/updating-contours-for-matplotlib-animation) which told me I can't update contours, I have to remove them instead.'

Nothing mind-blowing, but I always find it useful to find out how other people solve their problems. I cannot remember what I searched for when I was trying to solve the problem for myself, but I guess I must not have focussed my search on the key aspect of my plots that were causing the problems, namely, the contours.


### Goal for the future
To guage where I am at, I asked Tom how long it took them to create the pull request. The answer was roughly 30 minutes! For me, that is incredibly fast. It is encouraging to see what I can strive for and what I will be able to achieve with continued practice and experience.


## Conclusion
As you can see, I learnt a great deal from this experience. The main lesson of course is to learn from others. With enough effort, I probably could have arrived at a solution myself, however, it is much faster this way *and* I learnt a whole bunch of other little things on the side.


## Github Repository
Here is the link to the [Github repository](https://github.com/Lovkush-A/l1l2_regularisation) in case you're interested in looking at the full code and comparing things for yourself. 