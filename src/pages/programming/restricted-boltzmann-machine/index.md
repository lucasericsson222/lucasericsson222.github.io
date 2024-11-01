---
layout: '/src/layouts/BaseLayout.astro'
title: "Restricted Boltzmann Machines"
pubDate: 2024-10-10
description: 'All about Restricted Boltzmann Machines'
author: 'Lucas Ericsson'
---
<h1><span class="tokipona" lang="tok"> ilo toki pi nanpa tan nasa</span> Restricted Boltzmann Machines</h1>

*by Lucas Ericsson*

A Boltzmann Machine is a fully connected generative neural network,
where the probability of the network being in a specific state
follows the [Boltzmann Distribution](https://en.wikipedia.org/wiki/Boltzmann_distribution).
In contrast to linear or feed-forward nueral networks, which use layers of neurons
in order from input to output, the Boltzmann Machine initializes some nuerons to the data,
and lets those nuerons affect adjacent nuerons, repeatingly propogating in every direction.
This design allows it to learn distributions of unlabeled data 
and generates samples from those distributions.

<aside>
This page functions as notes for my future self and anyone else who happens to stumble upon here.
</aside>

The Boltzmann Distribution is defined as

<span class="katex-display">

$$p_i \propto \exp(-\frac{E(\sigma)}{kT})$$

</span>

Where $T$ is the temperature, and $E(\sigma)$ is the energy function of a state $\sigma$.

<aside>
    Temperature is a hyperparameter (non-learnable parameter) which controls the "randomness" of the model. A higher temperature means that the model will output a wider collection of results.
</aside>

For a Boltzmann Machine, the energy is defined as

<span class="katex-display">

$$E(\sigma) = - \displaystyle\sum_{i,j} \sigma_i \sigma_j w_{ij} - \sum_i b_i \sigma_i$$

</span>

where $i \ne j$, $w_{ij}$ is the weight of the edge between two nodes/vertices $i$ and $j$, and $b_i$ is the bias of vertex $i$.

Each neuron node has a binary value of either 0 or 1. Some other designs use -1/1 as the states, but these are interchangeable with some slight modifications to the sampling algorithm. 
There have been experiements into [Guassian distributed boltzmann machines](https://arxiv.org/pdf/1701.03647), but I will be only covering the Bernoulli distributed version.

From this definition, we can derive a method for this network to learn any distribution of data,
whether it be the inputs and outputs of an AND gate, or the pixels of handwritten numbers (MNIST).

## Restricted Boltzmann Machine

The Restricted Boltzmann Machine "restricts" the type of graph being used as the neural network to a bipartite graph of visible and hidden nuerons.

This changes the energy formula to be something like:

<span class="katex-display">

$$E(v, h) = - \displaystyle \sum_{i,j} v_i h_j w_{ij} - \sum_i b_i v_i - \sum_i c_i h_i$$

</span>

This restriction will make more sense when we talk about sampling from the Boltzmann machine.

# Sampling

Calculating the probability of a specific state is infeasable:
in order to calculate $p(\sigma)$ one would have to calculate the values of $\exp\left(-\frac{E(\sigma)}{kT}\right)$ for every value of $\sigma$ to normalize the probability distribution.

That's the hidden problem with $\propto$, the actual distribution is:

<span class="katex-display">

$$\displaystyle p_i = \frac {\exp\left(-\frac{E(\sigma)}{kT}\right)}{Z}$$

</span>

where $Z=\displaystyle \sum_\sigma\exp\left(-\frac{E(\sigma)}{kT}\right)$

$Z$ is sometimes called the "Partition Function" but it is intractable, as the number of items to add grows exponentially with the size of the state dimension.

If we are using a Restricted Boltzmann Machine, we can use [Gibb's Block Sampling](https://en.wikipedia.org/wiki/Gibbs_sampling), a [Monte Carlo Markov Chain](https://www.deeplearningbook.org/contents/monte_carlo.html) method, in order to draw samples from the distribution without calculating the partition function.

In an RBM, we have to layers of neurons and each nueron is only dependent on input from neurons in the opposing layer. This means that we have a set up of independent "blocks" of neurons.

If we can calculate $p(h_i = 1|v)$ then we can probabilistically flip the value of $h$. This will result in the hidden nuerons being distributed according to $h|v$.
Since the shape of the RBM is symmetric, if we can calculate $p(h_i = 1|v)$ then we can calculate $p(v_i = 1|h)$.

So, now we can essentially do "hops" to and from the hidden nuerons like so:

<span class="katex-display">

$$P(h|v) \rightarrow P(v'|h) \rightarrow P(h'|v') \rightarrow \dots$$

</span>

By defining this as a Markov Chain, we can show that this eventually converges to $P(v,h)$ which is the probability distribution we were having trouble calculating values from in the first place.

Since the Restricted Boltzmann Machine visible nuerons are conditionally independent from each other given the hidden nuerons, we are able to batch-process all of the visible neurons in one block and all of the hidden neurons in one block.

[A RBM paper by Asja Fischer and Christian Igel](http://cms.dm.uba.ar/academico/materias/1ercuat2018/probabilidades_y_estadistica_C/5a89b5075af5cbef5becaf419457cdd77cc9.pdf) shows that

<span class="katex-display">

$$P(h_i = 1|v)=\sigma(\sum_j w_{ij}v_j + c_i)$$

</span>

<span class="katex-display">

$$P(v_j = 1|h)=\sigma(\sum_i w_{ij}h_i + b_j)$$

</span>

where $\sigma(z) = \frac{1}{1+e^{-z}}$ is the sigmoid function.

Some psuedocode for the Gibb's sampling might look like:

```python
import numpy as np
rbm = RBM()
# rbm.weights is a matrix where the columns 
# correspond to the hidden neuron is connected to, 
# and the rows to the visible nuerons
v = normal(0, 1, rbm.visible_biases.shape)
h = normal(0, 1, rbm.hidden_biases.shape)
while (sampling):
    # visible -> hidden
    dist = sigmoid(
        (rbm.hidden_biases + (v.T @ rbm.weights).T ) 
        / rbm.temp)
    unif = np.random.rand(dist.shape)
    h = (unif <= dist).astype(int)

    # hidden -> visible
    dist = sigmoid(
        (rbm.visible_biases + rbm.weights @ h)
         / rbm.temp)
    unif = np.random.rand(dist.shape)
    v (unif <= dist).astype(int)

# now (v,h) ~ p(v,h)
```

Note that the hidden nuerons are being updated all at the same time (and likewise the visible neurons).
This is the significant reason why Restricted Boltzmann Machines are commonly used more than their unrestricted counterpart.

Instead of having just a forward and a backward pass, a Boltzmann Machine would have to step through each neuron individually and update it depending on it's neighbors.

# Training

to be continued...