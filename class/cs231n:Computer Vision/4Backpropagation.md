### BackPropagation

Contents:

- [Introduction](https://cs231n.github.io/optimization-2/#intro)
- [Simple expressions, interpreting the gradient](https://cs231n.github.io/optimization-2/#grad)
- [Compound expressions, chain rule, backpropagation](https://cs231n.github.io/optimization-2/#backprop)
- [Intuitive understanding of backpropagation](https://cs231n.github.io/optimization-2/#intuitive)
- [Modularity: Sigmoid example](https://cs231n.github.io/optimization-2/#sigmoid)
- [Backprop in practice: Staged computation](https://cs231n.github.io/optimization-2/#staged)
- [Patterns in backward flow](https://cs231n.github.io/optimization-2/#patterns)
- [Gradients for vectorized operations](https://cs231n.github.io/optimization-2/#mat)
- [Summary](https://cs231n.github.io/optimization-2/#summary)

#### Introduction

**backpropagation**, which is a way of computing gradients of expressions through recursive application of **chain rule**.

#### Simple expressions, interpreting the gradient

![image-20230927202125738](img/image-20230927202125738.png)

#### Compound expressions, chain rule, backpropagation

#### Intuitive understanding of backpropagation

Every gate in a circuit diagram gets some inputs and can right away compute two things: 1. its output value and 2. the *local* gradient of its output with respect to its inputs. 

*This extra multiplication (for each input) due to the chain rule can turn a single and relatively useless gate into a cog in a complex circuit such as an entire neural network.*

#### Modularity: Sigmoid example

#### Backprop in practice: Staged computation

#### Patterns in backward flow

#### Gradients for vectorized operations