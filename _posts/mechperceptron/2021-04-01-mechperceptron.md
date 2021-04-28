---
layout: post
title: "Mechanical Perceptron"
excerpt: "An exploration of the physical history behind perceptrons."
categories: [reinforcement learning]
comments: true
image:
    feature: her/cover.png
---

* generated ToC
{:toc}

## What is the Perceptron?

If you’ve dabbled in machine learning, you’ve probably heard about the perceptron in some form. Whether it’s in an introduction to ML course or the result of the google search “what is the P in MLP,” they seem to be everywhere. As a fundamental building block of modern machine learning, the perceptron rightfully has its place in the annals of ML history. 

{% include image_set_single_caption.html urls="/img/mechperceptron/mlp.png, /img/mechperceptron/mlpml.png" description="Make sure you tack 'machine learning' onto your google search!" size=100 %}

Today, you can write the perceptron in less than 20 lines of code. But it was not always this easy: when the perceptron was initially conceived, it was a purely hardware construct (which you'll get to see later on).

We may take the perceptron for granted today, but it was groundbreaking at its inception: designed to mimic neurons firing together, it gave the general public hope that machines would one day think and reason as humans do.

{% include image.html url="/img/mechperceptron/nyt.png" description="A clipping from a NYT article about the perceptron when it was first unveiled in 1958." size=50 %}

In its simplest form, the perceptron can be used to learn binary classifiers for linearly separable data in a supervised learning setting. It does this by finding the optimal weights for a linear classifier (i.e. a line that separates two classes). By paying attention to misclassified datapoints, the perceptron is able to find the optimal weights for a linear classifier such that it divides two classes.

To understand this better, let's see it in action.

### A Soft Introduction to Perceptrons

Let's look at a binary classification task: consider a toy example where we want to find the equation of a line that separates the blue points from the red points. We know the blue points can be separated from the red points by a line:

{% include image.html url="/img/mechperceptron/perceptron1_unlabelled.png" description="We want to separate the blue points from the red, but this line isn't going to cut it." size=50 %}

Well… maybe not this line, since the blue point at (1, 3) is on the same side of the line as the red point. However, we know that we can update the slope and intercept of this line so that it _does_ separate the red from the blue. Specifically, since we know that the blue point at (1, 3) is on the wrong side of the line, we can update the line’s slope and intercept so that this blue point falls to the right side of the line:

{% include image.html url="/img/mechperceptron/perceptron2_unlabelled.png" description="This line looks like it's doing better than the last one!" size=50 %}

So now we have that the red points are on the left side of the line, and the blue points are on the right side of the line. But this toy example raises an interesting question: is there a systematic way to update the slope and intercept, or weights, of the linear separator based on misclassified points?

This is the core idea behind perceptron: We use classification mistakes we make during training to inform the weight updates to our linear classifier. And we can do this algorithmically.

### The Algorithm

{% include image.html url="/img/mechperceptron/perceptronpseudocode.png" description="Perceptron in pseudocode." size=100 %}

This groundbreaking algorithm to determine the equation of a line separating two classes is less than 20 lines--huzzah!

At a high level, the algorithm iterates through each datapoint $$(\mathbf{x_i}, y_i)$$ and checks if the line parametrized by weights $$\mathbf{w}$$ classifies $$\mathbf{x_i}$$ correctly into class $$y_i$$ using the dot product (lines 3-6). If  $$\mathbf{x_i}$$ has been misclassified, then the weights are updated based on $$\mathbf{x_i}$$, its true class $$y_i$$, and learning rate $$\alpha$$. We assume that class above the line is denoted by 1, while the class below the line is denoted by -1. Let’s unpack what this all means in the next section.

### Why Does the Dot Product Work?

Earlier, in our toy example, we talked about updating the parameters of the equation of the linear separator defined in slope-intercept form. But lines can also be defined in standard form, or $$Ax_1 + Bx_2 + C = 0$$ (and this is actually generalizable to higher dimensions, which we like). So you can think of the weights as these coefficients $$A, B, \text{ and } C$$, and our data point $$\mathbf{x_i}$$ as the vector $$[x_1, x_2, 1]$$. The dot product between the two is, by definition, $$Ax_1 + Bx_2 + C$$.

Notice that the equation of a line is defined as $$Ax_1 + Bx_2 + C = 0$$. Equivalently, $$\mathbf{w}^T \cdot \mathbf{x_i} = 0$$. Hmm… it looks like we’ll need to remember some properties of the dot product. Namely:

* $$\mathbf{w}^T \cdot \mathbf{x_i} = 0$$ means that vector $$\mathbf{w}$$ is perpendicular to $$\mathbf{x_i}$$. 
* $$\mathbf{w}^T \cdot \mathbf{x_i} < 0$$ means that the angle between $$\mathbf{w}$$ and $$\mathbf{x_i}$$ is obtuse. 
* $$\mathbf{w}^T \cdot \mathbf{x_i} > 0$$ means that the angle between $$\mathbf{w}$$ and $$\mathbf{x_i}$$ is acute. 

This now gives better insight into line 6 of the algorithm above: Being on one side of the line or the other corresponds to forming an acute or obtuse angle with the normal to the line!

{% include image.html url="/img/mechperceptron/dotproduct.png" description="When a point lies above the line, it forms an acute angle with its normal. Likewise, when it lies below the line, it forms an obtuse angle with its normal." size=50 %}

The multiplication with $$y_i$$ in line 6 is just to make sure we have classified the point correctly: If $$y_i$$ is 1, this means the point should be classified above the line, so as long as the dot product of $$\mathbf{w}$$ and $$\mathbf{x_i}$$ is greater than 0, we have classified it correctly. If $$y_i$$ is -1, this means the point should be classified below the line, so as long as the _negative_ of the dot product of $$\mathbf{w}$$ and $$\mathbf{x_i}$$ is greater than 0, we have classified it correctly.

### Why does the update rule work?

$$\mathbf{w} = \mathbf{w} + \alpha \cdot y_i \cdot \mathbf{x_i}$$

Okay, so the dot product can tell us which side of the line we’re on. But how should we update the line when we make a misclassification?

Notice that line 1 defines each class as -1 and 1, respectively. Let’s assume that class 1 is red and class -1 is blue from the toy example:

{% include image.html url="/img/mechperceptron/perceptron1.png" description="" size=50 %}

$$\mathbf{w}$$ starts out as $$[-1, 1, 0]$$. When dotted with the red point at $$(1, 4)$$, we get that $$\mathbf{w}^T \cdot \mathbf{x} = (-1 * 1) + (1 * 4) + (0 * 1) = 3$$. This is greater than zero, so we continue. 

However, we know that the blue point at $$(1, 3)$$ is misclassified. But how does the perceptron algorithm know that? $$\mathbf{w}^T \cdot \mathbf{x} = (-1 * 1) + (1 * 3) + (0 * 1) = 2$$. This is also greater than zero, but remember that we actually want this point to be classified below the line!

At a high level, we want to increase the slope and intercept of this line proportional to the misclassified point. If the line in standard form is $$Ax_1 + Bx_2 + C = 0$$, then the line in slope intercept form is $$x_2 = -\frac{A}{B}x_1 - \frac{C}{B}$$. This means that in order to increase the line’s slope and intercept, we need to _subtract_ from $$\mathbf{w_i} = [A, B, C]$$. This is what line 7 accomplishes: since $$y_i$$ is -1, we end up subtracting $$\mathbf{x_i}$$ from $$\mathbf{w}$$. The learning rate $$\alpha$$ is simply a constant (e.g. 0.2) that scales the weight update at line 7 so that we don’t change the line too dramatically. 

{% include image.html url="/img/mechperceptron/perceptron2.png" description="" size=50 %}

Great, now the points are classified properly! Pretty neat, huh?

## A Brief History of the Perceptron

### Neurons

{% include image.html url="https://rewardfoundation.org/wp-content/uploads/2016/02/Screen-Shot-2017-08-24-at-10.50.36-758x245.png" description="Neural activity inspired AI research in the 20th century." size=100 %}

But what is the origin of the perceptron? We first need to talk about <a href="https://nautil.us/issue/21/information/the-man-who-tried-to-redeem-the-world-with-logic" target="_blank">McCulloch and Pitts:</a> : in 1943, they published their groundbreaking paper, <a href="https://www.cs.cmu.edu/~./epxing/Class/10715/reading/McCulloch.and.Pitts.pdf" target="_blank">“A Logical Calculus of Ideas Immanent in Nervous Activity.”</a> They were inspired by the activity of neurons: a neuron only fires when the “sum” of electrical activity input to the neuron reaches a certain threshold. 

{% include image.html url="/img/mechperceptron/mcp.png" description="The incoming arrows to the x nodes demonstrates that the output of one neuron acts as the input to another neuron. g represents the summation, and f represents thresholding." size=50 %}

Hopefully, you should sense some similarities with the previous section: in our perceptron algorithm from above, we classified a point as being above the line only when the weighted sum of the input point $$\mathbf{x_i}$$ was greater than our threshold 0, and below the line otherwise.

Although there were some limitations to the McCulloch-Pitts (MCP) neuron, it lay the groundwork for many other neuron-based units. And in 1957, our man Frank Rosenblatt would use the MCP neuron as the foundation for... the Perceptron. 

### Frank Rosenblatt and the Perceptron

{% include image.html url="https://news.cornell.edu/sites/default/files/styles/story_thumbnail_xlarge/public/2019-09/0925_rosenblatt_main.jpg?itok=SE0aS7ds" description="Frank Rosenblatt working on customized hardware for his perceptron." size=50 %}

In 1957, Rosenblatt published his seminal paper, “The Perceptron: A Perceiving and Recognizing Automaton.” He combined concepts from the MCP neuron, as well as other sources, in order to develop a working implementation of neuron-based learning for a basic image recognition task.

In it, he outlined the components of the perceptron we know and love. What is interesting is that he did not present a software algorithm to simulate this concept. Back then, computer simulation was non-trivial (no python 😢)... so he built a custom hardware implementation, using motorized potentiometers to control the weights of his classifier. 

{% include image.html url="https://2.bp.blogspot.com/-uLj0WAlG6to/Whr0WmYybsI/AAAAAAAAwvU/XjunlQIY36kRe9dZbf5Pm3xxx9dLXz8-wCLcBGAs/s1600/perceptron-smithsonian.jpg" description="The Mark I, Rosenblatt's first hardware implementation of the perceptron algorithm." size=50 %}

What a boppin' piece of history! Although the perceptron has since been dismantled, several components lie in the archives of museums across the US. In case anyone is interested, you can also read the original operating manual <a href="https://apps.dtic.mil/dtic/tr/fulltext/u2/236965.pdf" target="_blank">here</a>!

{% include image.html url="/img/mechperceptron/perceptronneuron.png" description="Note how similar this implementation is to the MCP neuron. Major differences include non-binary inputs and weighted sums." size=50 %}

The basic unit of Rosenblatt’s perceptron was a neuron composed of “sensory, association, and response” units. Respectively, this corresponded to the input, weighted sum, and thresholding/output units in the diagram above. Rosenblatt’s perceptron improved upon preexisting neurons, particularly the MCP neuron, by allowing non-binary inputs, computing the weights and thresholds algorithmically, and training via supervised methods.  Furthermore, Rosenblatt’s perceptron could handle multi-class classification.

{% include image.html url="/img/mechperceptron/sar.png" description="Sensory, association, and response units detailed in Rosenblatt's original paper." size=50 %}

If you want to understand more about Rosenblatt’s original hardware implementation of the perceptron, I highly recommend reading the <a href="https://blogs.umass.edu/brain-wars/files/2016/03/rosenblatt-1957.pdf" target="_blank">original paper</a>. Not only is it interesting to see the parallels between Rosenblatt’s hardware implementation and modern ML concepts, but it’s also a time capsule of academia in the 50s!

### The Perceptrons Controversy

When Rosenblatt first unveiled the perceptron, the attention was almost immediate. Shortly after the publication of “The Perceptron: A Perceiving and Recognizing Automaton,” the ONR (Office of Naval Research) held a press conference about Rosenblatt’s findings which sparked lots of buzz about the capabilities of his machine. In fact, the New York Times released <a href="https://forum.huawei.com/enterprise/en/perceptron/thread/624123-100504" target="_blank">an article</a> touting Rosenblatt’s work as nothing short of a miracle, citing wild claims that one day AI could self replicate, or be used as “mechanical space explorers”. 

{% include image.html url="https://media3.giphy.com/media/12R2bKfxceemNq/giphy.gif" description="There was a lot of buzz about mechanical space explorers." size=50 %}

Hopes were high for the perceptron and neural-based AI. At the same time, many other researchers were working on perceptrons as well, and Rosenblatt himself jumped to the SRI (Stanford Research Institute) to work on MINOS, a mechanical perceptron that used magnets and servos instead of motorized potentiometers. 

However, Rosenblatt was not without critics. While he acknowledged that the perceptron had limitations, he was also prone to overstating the power of single-layer neural nets. There was a flurry of research into single-layer nets at the time, since backpropagation had not been invented yet and there was no easy way to train multi-layer neural nets at the time. Arguably his greatest critic and rival, one of the father of AI, Minsky, and his colleague Papert, severely doubted the efficacy of the neural net approach to AI. As a result, they published a book in 1969 titled <a href="https://en.wikipedia.org/wiki/Perceptrons_(book)" target="_blank">“Perceptrons: an introduction to computational geometry”</a>. In it, they set up a single-layer perceptron under specific constraints and demonstrated its limitations. While it was backed by mathematical proofs, it came across as a major beat-down of the neural network approach to AI. 

{% include image.html url="/img/mechperceptron/kanyetaylor.png" description="Minsky and Papert's publication exemplified the great AI rivalry Minsky and Rosenblatt supposedly had." size=50 %}

To quote <a href="https://www.researchgate.net/publication/3081582_Review_of_%27Perceptrons_An_Introduction_to_Computational_Geometry%27_Minsky_M_and_Papert_S_1969" target="_blank">one review</a>, 

> I find a certain pleasant irony in the fact that Marvin Minsky and Seymour Papert, long vocal as vigorous opponents of the Perceptron approach to building “intelligent machines,” have given the concept a new interest and vigor by giving their essay on computational geometry the title “Perceptrons."

While their publication revealed important shortcomings to the neural network approach, critics felt that the constraints Minsky and Papert imposed on the perceptrons they evaluated were too stringent (and generally agreed that it was just an elaborate smack-down of Rosenblatt’s work). Despite the snippet from the review above, many credit “Perceptrons: an introduction to computational geometry” as the cause for an “AI winter” throughout the 70s and 80s because its publication, as well as Rosenblatt’s untimely death in 1971, drastically reduced interest in neuron-based approaches to artificial intelligence.

{% include image.html url="/img/mechperceptron/risky.png" description="Some say the publication induced an AI winter for two decades." size=50 %}

However, other sources say that other trends in AI were responsible for the shift away from neural networks and that AI historians are just drama queens who want to play up the beef between Rosenblatt and Minsky. In Mikel Olazaran's <a href="https://www.jstor.org/stable/285702?seq=11#metadata_info_tab_contents" target="_blank">"Official History of the Perceptrons Controversy"</a>, he notes that: 

> The official history conveniently exaggerates the phenomenon of the abandonment of neural nets. Although neural nets were largely rejected _as an approach to AI_, some ... continued working on neural nets ... Therefore the (inaccurate) view of the 'abandonment of neural nets' can be seen as legitimating the emergence of symbolic AI, rather than as an exact description of the result of the perceptrons controversy.

Either way, considering how integral perceptrons are to modern day machine learning, I’m sure Rosenblatt would be saying “I told you so” to Minsky right now!





