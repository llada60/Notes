### Neural Networks Part 1: Setting up the Architecture

Contents

- [Quick intro without brain analogies](https://cs231n.github.io/neural-networks-1/#quick)
- Modeling one neuron
  - [Biological motivation and connections](https://cs231n.github.io/neural-networks-1/#bio)
  - [Single neuron as a linear classifier](https://cs231n.github.io/neural-networks-1/#classifier)
  - [Commonly used activation functions](https://cs231n.github.io/neural-networks-1/#actfun)
- Neural Network architectures
  - [Layer-wise organization](https://cs231n.github.io/neural-networks-1/#layers)
  - [Example feed-forward computation](https://cs231n.github.io/neural-networks-1/#feedforward)
  - [Representational power](https://cs231n.github.io/neural-networks-1/#power)
  - [Setting number of layers and their sizes](https://cs231n.github.io/neural-networks-1/#arch)
- [Summary](https://cs231n.github.io/neural-networks-1/#summary)
- [Additional references](https://cs231n.github.io/neural-networks-1/#add)

#### Activation function

The non-linearity is where we get the *wiggle*.

- **sigmoid function** (logistic) $\sigma(x)$

$$
\sigma(x)=\frac{1}{1+e^{-x}}
$$

![image-20231226172924174](img/image-20231226172924174.png)

drawback

- **Sigmoids saturate and kill gradients**: when the neuron's activation saturates at either tail of 0 or 1, the gradient at these regions is almost zero. If the local gradient is very small, it will kill the gradient and almost no signal will flow through the neuron to its weights and recursively to its data. If the weights are too large then most neurons would become saturated and the network will barely learn.
- **Sigmoid outputs are not zero-centered.**: the data coming into a neuron is always positive, then the gradients on the weights during backpropagation become either all be positive or all negative (depending on the gradient of the whole expression). This could introduce zig-zagging dynamics in the gradient updates for the weights. However, once these gradients are added up across a batch of data the final update for the weights can have variable signs, somewhat mitigating this issue.

- **Tanh**

$$
tanh(x)=2\sigma(2x)-1
$$

its activations saturate, but unlike the sigmoid neuron its output is zero-centered.

- **ReLU**

$$
f(x)=max(0,x)
$$

![img](https://cs231n.github.io/assets/nn1/relu.jpeg)

- Pros
  - accelerate the convergence of stochastic gradient descent compared to the sigmoid/tanh function <-- due to linear, non-saturating form
  - compared to tanh/sigmoid that involve expensive operations, the ReLU can be implemented.
- Cons
  - ReLU units can be fragile during training and can "die". Some neurons will forever be zero after it died. Maybe caused by the *high learning rate*.

- **Leaky ReLU**
  $$
  f(x)=\mathbb{1}(x<0)(\alpha x)+\mathbb{1}(x\geq0)(x)
  $$

- **Maxout**

$$
max(w_1^Tx+b_1,w_2^Tx+b_2)
$$

where ReLU is its special case: $w_1,b_1=0$

unlike the ReLU neurons it doubles the number of parameters for every single neuron -> high total number of parameters.

**TLDR**: “*What neuron type should I use?*” Use the ReLU non-linearity, be careful with your learning rates and possibly monitor the fraction of “dead” units in a network. If this concerns you, give Leaky ReLU or Maxout a try. Never use sigmoid. Try tanh, but expect it to work worse than ReLU/Maxout.

#### Single neuron as a linear classfier

##### Binary Softmax classifier

$$
P(y_i=1|x_i;w)=\sigma(\sum_iw_ix_i+b)\\
P(y_i=0|x_i;w)=1-P(y_i=1|x_i;w)
$$

##### Binary SVM classifier

 attach a max-margin hinge loss to the output of the neuron and train it to become a binary Support Vector Machine.

##### Regularization interpretation

interpreted as *gradual forgetting*, since it would have the effect of driving all synaptic weights $w$ towards zero after every parameter update.

#### Neural Network architectures

##### Layer-wise Organization

*full-connected layer*: neurons between two adjacent layers are fully pairwise connected

*networks* = Artificial Neural Networks(ANN) = Multi-Layer Perceptrons (MLP)

*Output layer*: a linear identity activation function (an activation function x)

##### Representational power

the neural network can approximate any continuous function. （[*Approximation by Superpositions of Sigmoidal Function*](http://www.dartmouth.edu/~gvc/Cybenko_MCSS.pdf) from 1989 (pdf), or this [intuitive explanation](http://neuralnetworksanddeeplearning.com/chap4.html) from Michael Nielsen）

多层神经网络Pros：

1. 神经网络的模型的层数增加了，就提高了函数映射的非线性程度，即提高了模型的*Complexity*。

2. 加深的另一个好处是强化网络的抽象能力，让每层学习简单的内容，一层层的效果组合起来能实现更复杂的任务。

##### Setting number of layers and their sizes

**Overfitting**: when a model with <u>high capacity</u> fits the noise in the data instead of the (assumed) underlying relationship.

It models the data as two blobs and interprets the few red points inside the green cluster as **outliers** (noise). In practice, this could lead to better **generalization** on the test set.

![image-20230927224810169](img/image-20230927224810169.png)

##### Solutions for overfitting

1. Regularization
2. Big networks
3. Dropout
