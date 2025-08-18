

### linear Classification

Contents:

- Linear Classification
  - [Parameterized mapping from images to label scores](https://cs231n.github.io/linear-classify/#parameterized-mapping-from-images-to-label-scores)
  - [Interpreting a linear classifier](https://cs231n.github.io/linear-classify/#interpreting-a-linear-classifier)
  - Loss function
    - [Multiclass Support Vector Machine loss](https://cs231n.github.io/linear-classify/#multiclass-support-vector-machine-loss)
  - [Practical Considerations](https://cs231n.github.io/linear-classify/#practical-considerations)
  - [Softmax classifier](https://cs231n.github.io/linear-classify/#softmax-classifier)
  - [SVM vs. Softmax](https://cs231n.github.io/linear-classify/#svm-vs-softmax)
  - [Interactive web demo](https://cs231n.github.io/linear-classify/#interactive-web-demo)
  - [Summary](https://cs231n.github.io/linear-classify/#summary)
  - [Further Reading](https://cs231n.github.io/linear-classify/#further-reading)

Two major components of **linear classification**:

- **score function**: maps the raw data to class scores

- **loss function**: quantifies the agreement between the predicted scores and the ground truth labels.

#### Parameterized mapping from images to label scores

images $X_i \in R^D$ associated with a label $y_i,\ i=1...N\ and\ y_i\in1...K$

score function $f:R^D\rightarrow R^K$

##### Linear Classifier

$$
f(x_i,W,b)=Wx_i+b\\
f:R^D\rightarrow R^K
$$

$W\in R^{K\times D}$: weights 

$b \in R^K$: bias vector

![image-20230926192808302](img/image-20230926192808302.png)

*Bias trick*: extending the vetor $x_i$ with 1 additional dimension that always holds the constant 1 - a default *bias dimension*

![image-20230926193353100](img/image-20230926193353100.png)



*Image Data Preprocessing*: <u>center your data</u> on zero mean.

#### Loss function

##### Multiclass Support Vector Machine (SVM) 

SVM loss is set up so that the correct class for each image to have a socre higher than the incorrect classes by some fixed margin $\Delta$

given the pixels of image $x_i$ and the label $y_i$ that specifies the index of the correct class. The socre function takes the pixels and computes the vector  $f(x_i, W)$ of class scores, which we will abbreviate to s, e.g., the socre for the j-th class is the j-th element $s_j = f(x_i,W)_j$
$$
L_i = \sum_{j\neq y_i}max(0, s_j-s_{y_i}+\Delta)
$$

###### hinge loss / max-margin loss

对同一张图片的k个类标签score值约束
$$
L_i=\sum_{j\neq y_i}max(0,s_j-s_{y_i}+\Delta)\\
=\sum_{j\neq y_i}max(0,w_j^Tx_i-w_{y_i}^Tx_i+\Delta)
$$

=> wants the score of the *correct class* $w_j$ to be larger than the *incorrect class* scores by at leat by $\Delta$

![image-20230926193915608](img/image-20230926193915608.png)

*squared hinge loss SVM / L2-SVM*
$$
max(0,-)^2
$$

###### Regularization

Pros: improve the generalization performance of the classifiers on test images and lead to less overfitting.

The <u>set of W is not necessarily unique</u>: there might be many similar **W** that correctly classify the examples.

e.g.1: One easy way to see this is that if some parameters **W** correctly classify all examples (so loss is zero for each example), then any multiple of these parameters λW where λ>1 will also give zero loss because this transformation uniformly stretches all score magnitudes and hence also their absolute differences. For example, if the difference in scores between a correct class and a nearest incorrect class was 15, then multiplying all elements of **W** by 2 would make the new difference 30.

e.g. 2: The most appealing property is that penalizing large weights tends to **improve generalization**, because it means that no input dimension can have a very large influence on the scores all by itself. Suppose input vector x=[1,1,1,1], w1=[1,0,0,0] and w2 = [0.25, 0.25, 0.25, 0.25], then $w_1^Tx=w_2^Tx=1$ so both weight vectors lead to the same dot product, but the L2 penalty of w1 is 1 and of w2 is only 0.25. => L2 penalty prefers <u>smaller and more diffuse</u> weight vectors, the  final classifier is encouraged to take into account all input dimensions to small amounts rather than a few input dimensions and very strongly. => this can improve the <u>generalization</u> performance of the classifiers on test images and <u>lead to less overfitting.</u>

In other words, we wish to <u>encode some preference for a certain set of weights **W**</u> over others to remove this ambiguity.

**regularization penalty**             `

L1 norm (Lasso Regression): $\sum ||w||_1$

L2 norm (Ridge Regression): discourages large weights through an elementwise quadratic penalty over all parameters:                                        µ˚˚˚˚˚                                                                              
$$
R(W)=\sum_k\sum_lW_{k,l}^2
$$
full multiclass SVM loss= data loss + regularization loss
$$
L=\frac{1}{N}\sum_i L_i+\lambda R(W)
$$

#### Practical Considerations 

##### Setting Delta

it turns out that this hyperparameters can be safely be set to $\Delta = 1$ in all cases. Actually $\Delta$ and $\lambda$ control the same tradeoff between the data loss and the regularization loss in the objective (actually $\Delta$ hyperparameter is redundant).  The key to understanding this is that the <u>magnitude of the weights W has direct effect on the scores</u> (and hence also their differences): As we shrink all values inside W the score differences will become lower, and as we scale up the weights the score differences will all become higher. 

##### Binary Support Vector Machine

only had 2 classes
$$
L_i=C\ max(0,1-y_iw^Tx_i)+R(W)
$$
​		C: hyperparameter $\frac{1}{\lambda}$

​		$y_i\in\{-1,1\}$, $y_i=1$: correct; $y_i$=-1: incorrect

#### Softmax classifier

v.s. *binary logistic regression classifier*

softmax classifier is generalization to multiple classes.

##### softmax function

$$
f_j(z)=\frac{e^{z_j}}{\sum_k e^{z_{k}}}
$$

##### 交叉熵、熵、KL散度的关系

$$
H(p,q)=H(p)+D_{KL}(p||q)
\\ cross-entropy=inf-entropy+KL
$$

##### 1. entropy

表示随机变量不确定性的度量，熵越大，不确定性越大。
$$
H(p)=-\sum p(x)log(p(x))
$$

-log(p): p越大越确定，熵越小

##### 2. cross-entropy loss of multi class softmax classifier

刻画实际输出与期望输出的距离。交叉熵值越小，两个概率分布越接近。
$$
H(p,q)=-\sum p(x)log(q(x))
$$
*binary softmax classifier / logistic regression*：
$$
H(p,q)=-(p(x_1)logq(x_1)+(1-p(x_1))log(1-q(x_1)))
$$
special：数据量足够大时，最大似然估计==最小化cross-entropy

##### 3. KL divergence

计算模型分布与目标分布之间的差距
$$
D_{KL}(p||q)=\sum p(x)(logp(x)-logq(x))
$$
SPECIAL: 当目标分布已知且不变时（目标分布为常数），cross-entropy loss  == KL divergence between the two distributions

#### softmax loss- cross-entropy loss

$$
L_i=-log(\frac{e^{f_{y_i}}}{\sum_j e^{f_{j}}})
$$

minimizing the cross-entropy between the estimated class probabilities and the "true" distribution
$$
H(p,q)=-\sum_xp(x)log\ q(x)
$$

p: a "true" distribution = [0,...,1,...,0]

$q=\frac{e^{z_j}}{\sum_k e^{z_{k}}}$ estimate d class probabilities

**Probabilistic interpretation**: Maximum Likelihood Estimation (MLE)
$$
P(y_i|x_i;W)=\frac{e^{f_{y_i}}}{\sum_j e^{f_{j}}}
$$
issues: Numeric stability, f-=max(f), avoid too large due to the exponentials

##### Practical issues: Numeric stability

the intermediate terms $e^{f_{y_i}}$ and $\sum_j e^{f_j}$ may be very large due to the exponentials. Dividing large numbers can be numerically unstable, so it is important to use a normalization trick.
$$
\frac{e^{f{y_i}}}{\sum_j e^{f_j}} = \frac{Ce^{f_{y_i}}}{C\sum_je^{f_j}} = \frac{e^{f_{y_i}+logC}}{\sum_je^{f_j+logC}}
$$
$logC = -max_j f_j => f_j+logC \leq 0$

#### SVM v.s. Softmax

![image-20230926212813889](img/image-20230926212813889.png)

**Softmax** classifier is never fully happy with the scores it produces: <u>the correct class could always have a higher probability and the incorrect classes always a lower probability</u> and the loss would always get better. 

However, the **SVM** is <u>happy once the margins are satisfied</u> and it does not micromanage the exact scores beyond this constraint. 

This can intuitively be thought of as a feature: For example, a car classifier which is likely spending most of its “effort” on the difficult problem of separating cars from trucks should not be influenced by the frog examples, which it already assigns very low scores to, and which likely cluster around a completely different side of the data cloud.