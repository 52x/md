---
title: (机器学习) 深度学习与神经网络笔记-提升训练的方法
date: 2017-03-12 10:45:00
tags: [Machine Learning, Math, Deep Learning]
---

# Improving the way neural networks learn

> http://neuralnetworksanddeeplearning.com/chap3.html

## The cross-entropy cost function

### The problem of learning slowdown

Using quadratic cost function, the learning could be very slow. Consider a simple example where there's only 1 neuron.

![](http://neuralnetworksanddeeplearning.com/images/tikz28.png)

The quadratic cost function is given by

$$C = \frac{(y-a)^2}{2}$$

where a is the neuron's output. Using chain rule we get

$$\frac{\partial C}{\partial w} = (a-y) \sigma'(z)x$$

$$\frac{\partial C}{\partial b} = (a-y) \sigma'(z)$$

When the output $a$ is close to 0 or 1, $\sigma'(z)$ becomes very small, and this usually makes the gradients small. That's the origin of the learning slowdown.

<!--more-->

### Introducing the cross-entropy cost function

Consider a neuron with several input variables

![](http://neuralnetworksanddeeplearning.com/images/tikz29.png)

We define the cross-entropy cost function by

$$C = - \frac{1}{n} \sum_x [y \ln a^L + (1-y) \ln (1-a^L)]$$

> In a network with N outputs with one-hot style, the cost function would be

> $$C = - \frac{1}{n} \sum_{xj} [y_j \ln a_j^L + (1 - y_j) \ln (1 - a_j^L)]$$

With cross-entroy cost, we deduce the partial derivatives below

$$\frac{\partial C}{\partial a^L} = -\frac{1}{n} \sum_x (\frac{y}{a^L} - \frac{1-y}{1-a^L})$$

$$\frac{\partial a^L}{\partial w_j} = \sigma'(z) x_j$$

$$\frac{\partial C}{\partial w_j} = \frac{1}{n} \sum_x \frac{a^L - y}{a^L (1 - a^L)} \sigma'(z) x_j$$

$$\frac{\partial C}{\partial b} = \frac{1}{n} \sum_x \frac{a^L - y}{a^L (1 - a^L)} \sigma'(z)$$

Using equation $\sigma'(z) = \sigma(z) (1 - \sigma(z)) = a^L(1 - a^L)$ we get

$$\frac{\partial C}{\partial w_j} = \frac{1}{n}\sum_x(a^L - y)x_j$$

$$\frac{\partial C}{\partial b} = \frac{1}{n}\sum_x(a^L - y)$$

It tells us that the rate at which the weight learns is controlled by the error of in the output. Thus the learning slowdown could be avoided.

## Softmax

### The softmax function

Softmax is an alternative approach to the problem of learning slowdown. The idea is to define a new type of output layer. Instead apply sigmoid function to get the output, we apply *softmax function*. The activation of the *j*th output neuron is

$$a_j^L = \frac{e^{z_j^L}}{\sum_k e^{z_k^L}}$$

The output fromt he softmax layer sum up to 1 thus can be thought of as a probability distribution.

### Using softmax and log-likelihood cost

The log-likelihood cost

$$C \equiv - \sum_j I(y_j=1) \ln a_j^L$$

With the cost function above, we can deduce the partial derivatives

$$\frac{\partial C}{\partial a_i^L} = - \frac{I(y_i=1)}{a_i^L}$$

$$\frac{\partial a_i^L}{\partial z_k} = \frac{I(i=k)e^{z_k} \sum - e^{z_k} e^{z_i}}{\sum^2} = I(i=k)a_i^L - a_k^L a_i^L$$

$$\frac{\partial C}{\partial z_k} = I(y_i=1)a_k^L - I(i = k)I(y_i = 1)$$

We only need the *i*th output where $y_i = 1$ to get derivative of each $k$, so we pick the $i$ so that $y_i = 1$, thus

$$\frac{\partial C}{\partial z_k} = a_k^L - I(y_k = 1) = a_k^L - y_k$$

$$\frac{\partial C}{\partial w_{jk}} = (a_k^L - y_k) a_j^{L-1}$$

$$\frac{\partial C}{\partial b_k} = a_k^L - y_k$$

These equations are the same as the analogous expressions obtained in the analysis of cross-entropy.

> Given this similarity, should you use a sigmoid output layer and cross-entropy, or a softmax output layer and log-likelihood? In fact, in many situations both approaches work well.

## Overfitting and regularization

### L2 Regularization

The idea of L2 regularization is to add an extra term controlling the magnitude of weights

$$C = C_0 + \frac{\lambda}{2n} \sum_w w^2 = - \frac{1}{n} \sum_{xj} [y_j \ln a_j^L + (1 - y_j) \ln (1 - a_j^L)] + \frac{\lambda}{2n} \sum_w w^2$$

where $C_0$ is the original cross-entropy loss. The partial derivatives become

$$\frac{\partial C}{\partial w} = \frac{\partial C_0}{\partial w} + \frac{\lambda}{n} w$$

$$\frac{\partial C}{\partial b} = \frac{\partial C_0}{\partial b}$$

So the gradient descent will be

$$b \rightarrow b - \eta \frac{\partial C_0}{\partial b}$$

$$w \rightarrow (1 - \frac{\eta \lambda}{n} w) - \eta \frac{\partial C_0}{\partial w}$$

where weight $w$ get rescaled, which is referred to as *weight decay*.

> Why regularization works. Small weights means that the behavior of the network won't change too much if we change a few random inputs.  That makes it difficult for a regularized network to learn the effects of local noise in the data. In a nutshell, regularized networks are constrained to build relatively simple models based on patterns seen often in the training data, and are resistant to learning peculiarities of the noise in the training data.

### L1 Regularization

$$C = C_0 + \frac{\lambda}{n} \sum_w \vert w \vert$$

The partial derivative on weight is

$$\frac{\partial C}{\partial w} = \frac{\partial C_0}{\partial w} + \frac{\lambda}{n} sgn(w)$$

The update rule in SGD on weight is

$$w \rightarrow (1 - \frac{\eta \lambda}{n} sgn(w)) - \eta \frac{\partial C_0}{\partial w}$$

L2 decay is proportional to $w$, while in L1 the weights shrink by a constant amount toward 0. When weights are small, L1 regularization shrinks more the L2, while when weights are big, the shrink is smaller the L2. 

The net result is that L1 regularization tends to concentrate the weight of the network in a relatively small number of high-importance connections, while the other weights are driven toward zero.

### Dropout

We start by randomly (and temporarily) deleting half the hidden neurons in the network, while leaving the input and output neurons untouched. We forward-propagate the input $x$ through the modified network, and then backpropagate the result, also through the modified network. After doing this over a mini-batch of examples, we update the appropriate weights and biases. We then repeat the process, first restoring the dropout neurons, then choosing a new random subset of hidden neurons to delete, estimating the gradient for a different mini-batch, and updating the weights and biases in the network.

![](http://neuralnetworksanddeeplearning.com/images/tikz31.png)

Support the proportion of dropout is $p$, because we always use $1-p$ of the weights to learn. To compensate for that, we multiply the weights by $1-p$ after learning is complete.

The idea behind dropout is ensembling.

## Weight initialization

Consider a single hidden neuron, who has $n_{in}$ input neurons from the previous layer

![](http://neuralnetworksanddeeplearning.com/images/tikz32.png)

when we initialize the weights with $N(0,1)$ Gaussian distribution. The score of the hidden neuron $z = \sum_j w_j x_j + b$ turns out to have very big variance. In particular, it's quitely likely that $\vert z \vert$ will be pretty large. If that's the case then the output $\sigma(z)$ from the hidden neuron will be very close to either 1 or 0. That means our hidden neuron will have saturated. And when that happens, as we know, making small changes in the weights will make only absolutely miniscule changes in the activation of our hidden neuron.

To prevent learning slowdown, we initialize the weights with $N(0, 1/\sqrt n_{in})$ so the score $z$ has standard deviation of 1. Empirically the learning speed will be increased substantially

![](http://neuralnetworksanddeeplearning.com/images/weight_initialization_30.png)

## Hyper-tuning

Broad strategy

- Start training a simple network on smaller validation set, the idea is to get feedback quickly
- When we increase the complexity of the network, the parameters need to be tuned again
- Use early stopping to determine the number of training epochs
- Use bigger learning rate and then switch to smaller learning rate to converge better
- Start with no regularization, and add a smaller $\lambda$, and increase by factors of 10
- Batch size is a relatively independent parameter

## Other techniques

### Momentum-based gradient descent

We introduce velocity variables $v = v_1, v_2, ...$, one for each corresponding $w_j$ variable. Then replace the gradient descent update rule $w \rightarrow w - \eta \nabla C$ by

$$v \rightarrow v' = \mu v - \eta \nabla C$$

$$w \rightarrow w' = w + v'$$

With momentum, the update is a modified gradient with shrinked previous gradient. That means that if the gradient is in (roughly) the same direction through several rounds of learning. 

The *momentum co-efficient* $\mu$ controls the friction. In practice, the momentum technique is commonly used, and often speeds up learning.

### Activation functions

#### tanh

*tanh* neuron is a rescaled version of the sigmoid function

$$tanh(z) \equiv \frac{e^z - e^{-z}}{e^z + e^{-z}}$$

$$\sigma(z) = \frac{1 + tanh(z/2)}{2}$$

tanh and sigmoid has the same shape but the output range is (-1, 1) instead of (0, 1). 

Indeed, for many tasks the tanh is found empirically to provide only a small or no improvement in performance over sigmoid neurons. 

#### ReLU

The *rectified linear unit*

$$max(0, w \cdot x + b)$$

ReLU has been proved better in image recognition problem. However, as with tanh neurons, we do not yet have a really deep understanding of when, exactly, rectified linear units are preferable, nor why. In a nutshell, the choice of activation function is rather empirical than theoretical.

> Sigmoid and tanh suffer from learning slowdown when the weighted input is closed to 0 or 1. By contrast, increasing the weighted input to a rectified linear unit will never cause it to saturate, and so there is no corresponding learning slowdown. On the other hand, when the weighted input to a rectified linear unit is negative, the gradient vanishes, and so the neuron stops learning entirely. 

## Code with regularization


```python
"""network2.py
~~~~~~~~~~~~~~

An improved version of network.py, implementing the stochastic
gradient descent learning algorithm for a feedforward neural network.
Improvements include the addition of the cross-entropy cost function,
regularization, and better initialization of network weights.  Note
that I have focused on making the code simple, easily readable, and
easily modifiable.  It is not optimized, and omits many desirable
features.

"""

#### Libraries
# Standard library
import json
import random
import sys

# Third-party libraries
import numpy as np


#### Define the quadratic and cross-entropy cost functions

class QuadraticCost(object):

    @staticmethod
    def fn(a, y):
        """Return the cost associated with an output ``a`` and desired output
        ``y``.

        """
        return 0.5*np.linalg.norm(a-y)**2

    @staticmethod
    def delta(z, a, y):
        """Return the error delta from the output layer."""
        return (a-y) * sigmoid_prime(z)


class CrossEntropyCost(object):

    @staticmethod
    def fn(a, y):
        """Return the cost associated with an output ``a`` and desired output
        ``y``.  Note that np.nan_to_num is used to ensure numerical
        stability.  In particular, if both ``a`` and ``y`` have a 1.0
        in the same slot, then the expression (1-y)*np.log(1-a)
        returns nan.  The np.nan_to_num ensures that that is converted
        to the correct value (0.0).

        """
        return np.sum(np.nan_to_num(-y*np.log(a)-(1-y)*np.log(1-a)))

    @staticmethod
    def delta(z, a, y):
        """Return the error delta from the output layer.  Note that the
        parameter ``z`` is not used by the method.  It is included in
        the method's parameters in order to make the interface
        consistent with the delta method for other cost classes.

        """
        return (a-y)


#### Main Network class
class Network(object):

    def __init__(self, sizes, cost=CrossEntropyCost):
        """The list ``sizes`` contains the number of neurons in the respective
        layers of the network.  For example, if the list was [2, 3, 1]
        then it would be a three-layer network, with the first layer
        containing 2 neurons, the second layer 3 neurons, and the
        third layer 1 neuron.  The biases and weights for the network
        are initialized randomly, using
        ``self.default_weight_initializer`` (see docstring for that
        method).

        """
        self.num_layers = len(sizes)
        self.sizes = sizes
        self.default_weight_initializer()
        self.cost=cost

    def default_weight_initializer(self):
        """Initialize each weight using a Gaussian distribution with mean 0
        and standard deviation 1 over the square root of the number of
        weights connecting to the same neuron.  Initialize the biases
        using a Gaussian distribution with mean 0 and standard
        deviation 1.

        Note that the first layer is assumed to be an input layer, and
        by convention we won't set any biases for those neurons, since
        biases are only ever used in computing the outputs from later
        layers.

        """
        self.biases = [np.random.randn(y, 1) for y in self.sizes[1:]]
        self.weights = [np.random.randn(y, x)/np.sqrt(x)
                        for x, y in zip(self.sizes[:-1], self.sizes[1:])]

    def large_weight_initializer(self):
        """Initialize the weights using a Gaussian distribution with mean 0
        and standard deviation 1.  Initialize the biases using a
        Gaussian distribution with mean 0 and standard deviation 1.

        Note that the first layer is assumed to be an input layer, and
        by convention we won't set any biases for those neurons, since
        biases are only ever used in computing the outputs from later
        layers.

        This weight and bias initializer uses the same approach as in
        Chapter 1, and is included for purposes of comparison.  It
        will usually be better to use the default weight initializer
        instead.

        """
        self.biases = [np.random.randn(y, 1) for y in self.sizes[1:]]
        self.weights = [np.random.randn(y, x)
                        for x, y in zip(self.sizes[:-1], self.sizes[1:])]

    def feedforward(self, a):
        """Return the output of the network if ``a`` is input."""
        for b, w in zip(self.biases, self.weights):
            a = sigmoid(np.dot(w, a)+b)
        return a

    def SGD(self, training_data, epochs, mini_batch_size, eta,
            lmbda = 0.0,
            evaluation_data=None,
            monitor_evaluation_cost=False,
            monitor_evaluation_accuracy=False,
            monitor_training_cost=False,
            monitor_training_accuracy=False):
        """Train the neural network using mini-batch stochastic gradient
        descent.  The ``training_data`` is a list of tuples ``(x, y)``
        representing the training inputs and the desired outputs.  The
        other non-optional parameters are self-explanatory, as is the
        regularization parameter ``lmbda``.  The method also accepts
        ``evaluation_data``, usually either the validation or test
        data.  We can monitor the cost and accuracy on either the
        evaluation data or the training data, by setting the
        appropriate flags.  The method returns a tuple containing four
        lists: the (per-epoch) costs on the evaluation data, the
        accuracies on the evaluation data, the costs on the training
        data, and the accuracies on the training data.  All values are
        evaluated at the end of each training epoch.  So, for example,
        if we train for 30 epochs, then the first element of the tuple
        will be a 30-element list containing the cost on the
        evaluation data at the end of each epoch. Note that the lists
        are empty if the corresponding flag is not set.

        """
        if evaluation_data: n_data = len(evaluation_data)
        n = len(training_data)
        evaluation_cost, evaluation_accuracy = [], []
        training_cost, training_accuracy = [], []
        for j in xrange(epochs):
            random.shuffle(training_data)
            mini_batches = [
                training_data[k:k+mini_batch_size]
                for k in xrange(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                self.update_mini_batch(
                    mini_batch, eta, lmbda, len(training_data))
            print "Epoch %s training complete" % j
            if monitor_training_cost:
                cost = self.total_cost(training_data, lmbda)
                training_cost.append(cost)
                print "Cost on training data: {}".format(cost)
            if monitor_training_accuracy:
                accuracy = self.accuracy(training_data, convert=True)
                training_accuracy.append(accuracy)
                print "Accuracy on training data: {} / {}".format(
                    accuracy, n)
            if monitor_evaluation_cost:
                cost = self.total_cost(evaluation_data, lmbda, convert=True)
                evaluation_cost.append(cost)
                print "Cost on evaluation data: {}".format(cost)
            if monitor_evaluation_accuracy:
                accuracy = self.accuracy(evaluation_data)
                evaluation_accuracy.append(accuracy)
                print "Accuracy on evaluation data: {} / {}".format(
                    self.accuracy(evaluation_data), n_data)
            print
        return evaluation_cost, evaluation_accuracy, \
            training_cost, training_accuracy

    def update_mini_batch(self, mini_batch, eta, lmbda, n):
        """Update the network's weights and biases by applying gradient
        descent using backpropagation to a single mini batch.  The
        ``mini_batch`` is a list of tuples ``(x, y)``, ``eta`` is the
        learning rate, ``lmbda`` is the regularization parameter, and
        ``n`` is the total size of the training data set.

        """
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        for x, y in mini_batch:
            delta_nabla_b, delta_nabla_w = self.backprop(x, y)
            nabla_b = [nb+dnb for nb, dnb in zip(nabla_b, delta_nabla_b)]
            nabla_w = [nw+dnw for nw, dnw in zip(nabla_w, delta_nabla_w)]
        self.weights = [(1-eta*(lmbda/n))*w-(eta/len(mini_batch))*nw
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
        delta = (self.cost).delta(zs[-1], activations[-1], y)
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

    def accuracy(self, data, convert=False):
        """Return the number of inputs in ``data`` for which the neural
        network outputs the correct result. The neural network's
        output is assumed to be the index of whichever neuron in the
        final layer has the highest activation.

        The flag ``convert`` should be set to False if the data set is
        validation or test data (the usual case), and to True if the
        data set is the training data. The need for this flag arises
        due to differences in the way the results ``y`` are
        represented in the different data sets.  In particular, it
        flags whether we need to convert between the different
        representations.  It may seem strange to use different
        representations for the different data sets.  Why not use the
        same representation for all three data sets?  It's done for
        efficiency reasons -- the program usually evaluates the cost
        on the training data and the accuracy on other data sets.
        These are different types of computations, and using different
        representations speeds things up.  More details on the
        representations can be found in
        mnist_loader.load_data_wrapper.

        """
        if convert:
            results = [(np.argmax(self.feedforward(x)), np.argmax(y))
                       for (x, y) in data]
        else:
            results = [(np.argmax(self.feedforward(x)), y)
                        for (x, y) in data]
        return sum(int(x == y) for (x, y) in results)

    def total_cost(self, data, lmbda, convert=False):
        """Return the total cost for the data set ``data``.  The flag
        ``convert`` should be set to False if the data set is the
        training data (the usual case), and to True if the data set is
        the validation or test data.  See comments on the similar (but
        reversed) convention for the ``accuracy`` method, above.
        """
        cost = 0.0
        for x, y in data:
            a = self.feedforward(x)
            if convert: y = vectorized_result(y)
            cost += self.cost.fn(a, y)/len(data)
        cost += 0.5*(lmbda/len(data))*sum(
            np.linalg.norm(w)**2 for w in self.weights)
        return cost

    def save(self, filename):
        """Save the neural network to the file ``filename``."""
        data = {"sizes": self.sizes,
                "weights": [w.tolist() for w in self.weights],
                "biases": [b.tolist() for b in self.biases],
                "cost": str(self.cost.__name__)}
        f = open(filename, "w")
        json.dump(data, f)
        f.close()

#### Loading a Network
def load(filename):
    """Load a neural network from the file ``filename``.  Returns an
    instance of Network.

    """
    f = open(filename, "r")
    data = json.load(f)
    f.close()
    cost = getattr(sys.modules[__name__], data["cost"])
    net = Network(data["sizes"], cost=cost)
    net.weights = [np.array(w) for w in data["weights"]]
    net.biases = [np.array(b) for b in data["biases"]]
    return net

#### Miscellaneous functions
def vectorized_result(j):
    """Return a 10-dimensional unit vector with a 1.0 in the j'th position
    and zeroes elsewhere.  This is used to convert a digit (0...9)
    into a corresponding desired output from the neural network.

    """
    e = np.zeros((10, 1))
    e[j] = 1.0
    return e

def sigmoid(z):
    """The sigmoid function."""
    return 1.0/(1.0+np.exp(-z))

def sigmoid_prime(z):
    """Derivative of the sigmoid function."""
    return sigmoid(z)*(1-sigmoid(z))
```
