### Optimization

contents:

- [Introduction](https://cs231n.github.io/optimization-1/#intro)
- [Visualizing the loss function](https://cs231n.github.io/optimization-1/#vis)
- Optimization
  - [Strategy #1: Random Search](https://cs231n.github.io/optimization-1/#opt1)
  - [Strategy #2: Random Local Search](https://cs231n.github.io/optimization-1/#opt2)
  - [Strategy #3: Following the gradient](https://cs231n.github.io/optimization-1/#opt3)
- Computing the gradient
  - [Numerically with finite differences](https://cs231n.github.io/optimization-1/#numerical)
  - [Analytically with calculus](https://cs231n.github.io/optimization-1/#analytic)
- [Gradient descent](https://cs231n.github.io/optimization-1/#gd)
- [Summary](https://cs231n.github.io/optimization-1/#summary)

#### Optimization

The process of finding the set of parameters W that minimize the loss function.

SVM loss is a convex problem.

#### Fllowing the Gradient

Two ways to compute the gradient: 

- A slow, approximate but easy way (**numerical gradient**)数值解
- A fast, exact but more error-prone way that requires calculus (**analytic gradient**).解析解

##### numerical gradient 数值解

$$
\frac{df(x)}{dx}=\lim_{h\rightarrow0}\frac{f(x+h)-f(x)}{h}
$$

##### analytic gradient 解析解

The numerical gradient is very simple to compute using the finite difference approximation, but the downside is that it is approximate (since we have to pick a small value of *h*, while the true gradient is defined as the limit as *h* goes to zero), and that it is very computationally expensive to compute. The second way to compute the gradient is analytically using Calculus, which allows us to derive a direct formula for the gradient (no approximations) that is also very fast to compute. However, unlike the numerical gradient it can be more error prone to implement, which is why in practice it is very common to compute the analytic gradient and compare it to the numerical gradient to check the correctness of your implementation. This is called a **gradient check**.

Gradient of SVM = respect to the row of W that corresponds to the correct class
$$
\nabla_{w_{y_i}}L_i=\mathbb{1}(w_j^Tx_i-w_{y_i}^Tx_i+\Delta>0)x_i
$$
when you’re implementing this in code you’d simply count the number of classes that didn’t meet the desired margin (and hence contributed to the loss function) and then the data vector $x_i$ scaled by this number is the gradient. 

#### Gradient Descent

the procedure of repeatedly evaluating the gradient and then performing a parameter update.

```python
while True:
  weights_grad = evaluate_gradient(loss_fun, data, weights)
  weights += - step_size * weights_grad # perform parameter update
```

**Stochastic Gradient Descent (SGD)** (on-line gradient descent): <u>single example</u> at a time to evaluate the gradient

**Minibatch gradient descent (MGD)** / **Batch gradient descent (BGD)**(now): to compute the gradient over batches of the training data.