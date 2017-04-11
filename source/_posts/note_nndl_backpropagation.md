---
title: (机器学习) 神经网络与深度学习笔记-反向传播
date: 2017-03-12 10:43:00
tags: [Machine Learning, Math, Deep Learning]
---

# Back Propagation

> http://neuralnetworksanddeeplearning.com/chap2.html

## Basic Notations

Use $w_{jk}^l$ to denote the weight for the connnection from the $k^{th}$ neuron in the $(l-1)^{th}$ layer to the $j^{th}$ neuron in the $l^{th}$ layer.

![](http://neuralnetworksanddeeplearning.com/images/tikz16.png)

Use $b_j^l$ for the bias for the $j^{th}$ neuron in the $j^{th}$ layer, and $a_j^l$ for the activation for the $j^{th}$ neuron in the $j^{th}$ layer. 

![](http://neuralnetworksanddeeplearning.com/images/tikz17.png)

<!--more-->

The forward propagation for a single activation

$$a_j^l = \sigma (\sum_k w_{jk}^l a_k^{l-1} + b_j^l)$$

Using Matrix notation. $w^l = (w_{jk}^l)$, $b_l = (b_j^l)$, $a_l = (a_j^l)$. The forward propagation could be rewrite

$$a^l = \sigma (w^l a^{l-1} + b^l)$$

We name the the input of activation function $z^l$ *weighted input*

$$z^l \equiv w^l a^{l-1} + b^l$$

## Assumptions of Cost Function

We use MSE as an example. The quadratic cost has the form

$$C = \frac{1}{2n} \sum_x \Vert y(x) - a^L(x) \Vert$$

where $n$ is the sample size. $y(x)$ is the training label, $L$ denotes the number of layers in the network, $a^L(x)$ is the vector of activations output from the network when x i the input.

The assumptions in order the BP can be applied,

- the cost function can be written as an average $C = 1/n \sum_{x} C_x$ over cost functions $C_x$ for individual training examples.
- the cost function can be written as a function of the outputs from the neural network.

## 4 Fundamental Equations of BP

Introduce the intermediate quantity $\delta_j^l$, which we call the *error* in the $j^{th}$ neuron in the $l^{th}$ layer. BP will give us a procedure to compute the error, and then will relate it to $\partial C / \partial w_{jk}^l$ and $\partial C / \partial b_j^l$. We define the error to be
|
$$\delta_j^l \equiv \frac{\partial C}{\partial z_j^l}$$

When the error is small, there's less room for us to adjust $\Delta z_j^l$ in order to make the cost $C$ smaller.

**An equation for the error in the output layer:**

single component

$$\delta_j^L = \frac{\partial C} {\partial a_j^L} \sigma' (z_j^L)$$

matrix-based form

$$\delta^L = \nabla_a C \odot \sigma' (z^L)$$

where $\nabla C_a$ is the vector whose components are the partial derivatives $\partial C / \partial a_j^L$. For quadratic cost function, $\nabla C_a = (a^L - y) \odot \sigma' (z_L)$ for a single sample.

**An equation for the error $\delta^l$ in terms of the error in the next layer $\delta^{l+1}$:**

single component form

$$\delta_j^l = \sum_k w_{kj}^{l+1} \delta_k^{l+1} \sigma' (z_j^l)$$

matrix-based form

$$\delta^l = ((w^{l+1})^T \delta^{l+1}) \odot \sigma' (z^l)$$

**An equation for the rate of change of the cost with respect of any bias in the network:**

single component

$$\frac{\partial C}{\partial b_j^l} = \delta_j^l$$

matrix-based form

$$\frac{\partial C}{\partial b} = \delta$$

**An equation for the rate of change of the cost with respect of any weight in the network:**

single component

$$\frac{\partial C}{\partial w_{jk}^l} = a_k^{l-1} \delta_j^l$$

matrix-based form

$$\frac{\partial C}{\partial w} = \delta_{out} a_{in}^T$$

> when the activation $a_{in}$ or error $\delta_{out}$ is small, the gradient term will also tend to be small. The weight will learn slow if either the input neuron is low-activation or if the output neuron has saturated, i.e., is either high- or low-activation. We might want an activation function $\sigma$ that never gets close to zero to prevent the slow-down of learning when sigmoid neurons saturate.

## The BP Algorithm

BP with SGD

1. Input a set of training samples
2. For each training sample $x$: set the corresponding input activation $a^{x,1}$, and perform the following steps:
 - Feedforward: for each l = 2, 3,..., L compute $z^{x,l} = w^l a^{x,l-1} + b^l$ and $a^{x,l} = \sigma(z^{x,l})$.
 - Ouput error $\delta^{x,L}$: compute the vector $\delta^{x,L} = \nabla_a C_x \odot \sigma'(z^{x, L})$.
 - Backpropagte the error: for each l = L - 1, L - 2,...,2 compute $\delta^{x,l} = ((w^{l+1})^T \delta^{x,l+1}) \odot \sigma'(z^{x,l})$.
3. Gradient descent: for each l = L, L - 1,..., 2 update the weights according to the rule $w^l \rightarrow w^l - \frac{\eta}{m} \sum_x \delta^{x,l}(a^{x,l-1})^T$, and the biases according to the rule $b^l \rightarrow b^l - \frac{\eta}{m} \sum_x \delta^{x,l}$.

## Code for BP


```python
"""
network.py
~~~~~~~~~~

A module to implement the stochastic gradient descent learning
algorithm for a feedforward neural network.  Gradients are calculated
using backpropagation.  Note that I have focused on making the code
simple, easily readable, and easily modifiable.  It is not optimized,
and omits many desirable features.
"""

#### Libraries
# Standard library
import random

# Third-party libraries
import numpy as np

class Network(object):

    def __init__(self, sizes):
        """The list ``sizes`` contains the number of neurons in the
        respective layers of the network.  For example, if the list
        was [2, 3, 1] then it would be a three-layer network, with the
        first layer containing 2 neurons, the second layer 3 neurons,
        and the third layer 1 neuron.  The biases and weights for the
        network are initialized randomly, using a Gaussian
        distribution with mean 0, and variance 1.  Note that the first
        layer is assumed to be an input layer, and by convention we
        won't set any biases for those neurons, since biases are only
        ever used in computing the outputs from later layers."""
        self.num_layers = len(sizes)
        self.sizes = sizes
        self.biases = [np.random.randn(y, 1) for y in sizes[1:]]
        self.weights = [np.random.randn(y, x)
                        for x, y in zip(sizes[:-1], sizes[1:])]

    def feedforward(self, a):
        """Return the output of the network if ``a`` is input."""
        for b, w in zip(self.biases, self.weights):
            a = sigmoid(np.dot(w, a)+b)
        return a

    def SGD(self, training_data, epochs, mini_batch_size, eta,
            test_data=None):
        """Train the neural network using mini-batch stochastic
        gradient descent.  The ``training_data`` is a list of tuples
        ``(x, y)`` representing the training inputs and the desired
        outputs.  The other non-optional parameters are
        self-explanatory.  If ``test_data`` is provided then the
        network will be evaluated against the test data after each
        epoch, and partial progress printed out.  This is useful for
        tracking progress, but slows things down substantially."""
        if test_data: n_test = len(test_data)
        n = len(training_data)
        for j in xrange(epochs):
            random.shuffle(training_data)
            mini_batches = [
                training_data[k:k+mini_batch_size]
                for k in xrange(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                self.update_mini_batch(mini_batch, eta)
            if test_data:
                print "Epoch {0}: {1} / {2}".format(
                    j, self.evaluate(test_data), n_test)
            else:
                print "Epoch {0} complete".format(j)

    def update_mini_batch(self, mini_batch, eta):
        """Update the network's weights and biases by applying
        gradient descent using backpropagation to a single mini batch.
        The ``mini_batch`` is a list of tuples ``(x, y)``, and ``eta``
        is the learning rate."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        for x, y in mini_batch:
            delta_nabla_b, delta_nabla_w = self.backprop(x, y)
            nabla_b = [nb+dnb for nb, dnb in zip(nabla_b, delta_nabla_b)]
            nabla_w = [nw+dnw for nw, dnw in zip(nabla_w, delta_nabla_w)]
        self.weights = [w-(eta/len(mini_batch))*nw
                        for w, nw in zip(self.weights, nabla_w)]
        self.biases = [b-(eta/len(mini_batch))*nb
                       for b, nb in zip(self.biases, nabla_b)]

    def backprop(self, x, y):
        """Return a tuple ``(nabla_b, nabla_w)`` representing the
        gradient for the cost function C_x.  ``nabla_b`` and
        ``nabla_w`` are layer-by-layer lists of numpy arrays, similar
        to ``self.biases`` and ``self.weights``."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        # feedforward
        activation = x
        activations = [x] # list to store all the activations, layer by layer
        zs = [] # list to store all the z vectors, layer by layer
        for b, w in zip(self.biases, self.weights):
            z = np.dot(w, activation)+b
            zs.append(z)
            activation = sigmoid(z)
            activations.append(activation)
        # backward pass
        delta = self.cost_derivative(activations[-1], y) * \
            sigmoid_prime(zs[-1])
        nabla_b[-1] = delta
        nabla_w[-1] = np.dot(delta, activations[-2].transpose())
        # Note that the variable l in the loop below is used a little
        # differently to the notation in Chapter 2 of the book.  Here,
        # l = 1 means the last layer of neurons, l = 2 is the
        # second-last layer, and so on.  It's a renumbering of the
        # scheme in the book, used here to take advantage of the fact
        # that Python can use negative indices in lists.
        for l in xrange(2, self.num_layers):
            z = zs[-l]
            sp = sigmoid_prime(z)
            delta = np.dot(self.weights[-l+1].transpose(), delta) * sp
            nabla_b[-l] = delta
            nabla_w[-l] = np.dot(delta, activations[-l-1].transpose())
        return (nabla_b, nabla_w)

    def evaluate(self, test_data):
        """Return the number of test inputs for which the neural
        network outputs the correct result. Note that the neural
        network's output is assumed to be the index of whichever
        neuron in the final layer has the highest activation."""
        test_results = [(np.argmax(self.feedforward(x)), y)
                        for (x, y) in test_data]
        return sum(int(x == y) for (x, y) in test_results)

    def cost_derivative(self, output_activations, y):
        """Return the vector of partial derivatives \partial C_x /
        \partial a for the output activations."""
        return (output_activations-y)

#### Miscellaneous functions
def sigmoid(z):
    """The sigmoid function."""
    return 1.0/(1.0+np.exp(-z))

def sigmoid_prime(z):
    """Derivative of the sigmoid function."""
    return sigmoid(z)*(1-sigmoid(z))
```
