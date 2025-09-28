[TOC]

## Autoencoder (AE)

![img](https://pic1.zhimg.com/v2-07a31d5e552f5da4f9bc0c5d21a3973e_1440w.jpg)

Decoder cannot use as generation model. Because vector space z is irregular.

## Variational Autoencoer (VAE)

![img](https://pic3.zhimg.com/v2-73bbc5125fc792a9bb759ff803a9b258_1440w.jpg)

约束z服从标准正态分布，则该z可直接用来实现图像生成。

### Intuition

1. Classic autoencoder that learns a set of cmopressed latent attribtes.
2. Use probabilistic distribution to model attribte range variation.
3. We can sample latent attributes to generate different images

![image-20250814185740972](img/image-20250814185740972.png)

### Evidence Lower Bound (ELBO)

## VQVAE

![img](https://picx.zhimg.com/v2-4382b984165170e238be55b914ff3503_1440w.jpg)

Image encode into **discrete vector**. (Intuition: 把图片编码成**离散向量**会更加自然。比如我们想让画家画一个人，我们会说这个是男是女; )

- Problem 1: 神经网络会默认输入满足一个连续的分布，而不善于处理离散的输入。如果你直接输入0, 1, 2这些数字，神经网络会默认1是一个处于0, 2中间的一种状态。
  - Solution 1: 它可以把每个输入单词都映射到一个独一无二的连续向量上。这样，每个离散的数字都变成了一个特别的连续向量了。

- Problem 2: 它不好采样。
  - Solution 2: PixelCNN能拟合一个离散的分布。比如对于图像，PixelCNN能输出某个像素的某个颜色通道取0~255中某个值的概率分布。VQ-VAE能把图像映射成一个「小图像」。我们可以把PixelCNN生成图像的方法搬过来，让PixelCNN学习生成「小图像」。这样，我们就可以用PixelCNN生成离散编码，再利用VQ-VAE的解码器把离散编码变成图像。

VQ-VAE 工作过程：

1. 训练VQ-VAE的编码器和解码器，使得VQ-VAE能把图像变成「小图像」，也能把「小图像」变回图像。
2. 训练PixelCNN，让它学习怎么生成「小图像」。
3. 随机采样时，先用PixelCNN采样出「小图像」，再用VQ-VAE把「小图像」翻译成最终的生成图像。

### encoder输出离散编码

假设嵌入空间已经训练完毕，对于编码器的每个输出向量$z_e(x)$，找出它在嵌入空间里的**最近邻**$z_q(x)$，把$z_e(x)$替换成$z_q(x)$作为解码器的输入。

### optimize encoder and decoder

**stop gradient**
$$
sg(x)=
\begin{cases}
x\ (in\ forward\ propagation)\\
0\ (in\ backwadr\ propagation)
\end{cases}
$$

$$
L_{reconstruct} = ||x-decoder(z_e(x)+sg(z_q(x)-z_e(x)))||_2^2
$$



```python
L = x - decoder(z_e + (z_q - z_e).detach())
```

### optimize embedding space

$$
L_e = ||z_e(x) - z_q(x)||_2^2
$$

$z_e(x)$ is the output from encoder; $z_q(x)$ is its nearest vector in the embedding space.

作者认为编码器和嵌入向量的学习速度应该不一样快， 引入$\beta$控制encoder的相对学习速度：
$$
L_e = ||sg(z_e(x)) - z_q(x)||_2^2+\beta ||z_e(x) - sq(z_q(x))||_2^2
$$
Total loss
$$
L = L_{reconstruct} + L_{embedding}
$$
