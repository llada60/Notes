# minGPT

https://github.com/karpathy/minGPT

[TOC]

## size

**block size (context length / sequence length)**: the maximum number of tokens the model can see at once in its input window. (==predict step)

> the maximum length is the last prediction's input length
>
> e.g.1: 
>
> output: x1x2...xn<EOS>
>
> block size: n
>
> e.g.2:
>
> output: x1x2...xn
>
> block size: n-1

**vocabulary size**: the size of the vocabulary, i.e. the total number of *distinct tokens* the model can represent and predict.

> e.g.: [digit addition] vocab_size=10 #0,1,...,9

token ID $\in$ [0, vocab_size - 1]

## 模型训练，定长数据输入

1. **拼接+定长切分**：将大量文本拼接成一个长序列，然后按固定长度（如 block_size=128、512、1024 等）滑窗切分。每个切片就是一个训练样本。这样每个 batch 的 seq length 固定，但原始文本长度不固定。
2. **<EOS> 标记**：有些任务会在每个句子或段落后加上特殊的 <EOS>（end of sequence）token，表示序列结束。这样模型可以学习到“哪里是结尾”。
3. **padding**：如果 batch 内样本长度不同，可以用 <PAD> token 补齐到同一长度。

```
如果 block_size=5，拼接后切分为：

[10, 20, 99, 30, 40] # 99 是 <EOS>
[50, 99, 60, 70, 80]
[99, <PAD>, <PAD>, <PAD>, <PAD>] # 最后一段不足5，pad补齐
```

## Embedding

`nn.Embedding` == `class Embedding(torch.nn.Module)`

- **Definition**: store word embeddings and retrieve them using indices. The input to the module is a list of indices, and the output is the corresponding word embeddings.

- **Attribute:** `weight (Tensor)`: the learnable weights of the module of shape `(num_embeddings, embedding_dim)`.

  ```
  import torch
  import torch.nn as nn
  
  embedding = nn.Embedding(num_embeddings=3, embedding_dim=5)  # 3个词，每个向量维度5
  idx = torch.tensor([0, 2])
  out = embedding(idx)
  ```

  - embedding.weight 是一个参数矩阵，形状 (num_embeddings, embedding_dim)，这里是 (3, 5)。

  - idx = [0, 2] 表示取第 0 和第 2 行向量。

  - out.shape = (2, 5)，就是拿到对应的 embedding 向量。

- **梯度的计算**

  在训练时，如果我们用 `out` 去算 loss 并反向传播：

  ```
  loss = out.sum()
  loss.backward()
  ```

  那么梯度只会更新 **被索引到的行**。
   比如上例中，只有 `embedding.weight[0]` 和 `embedding.weight[2]` 的 `.grad` 非零，其他行的梯度为 0。

  这是因为 embedding 的梯度计算等价于 **稀疏矩阵乘法**（只对用到的行做更新）。
   这点在大规模词表里非常高效。

## Sparse Tensor

**只存储非零元素**的张量数据结构。

- 节省存储空间 + 加速计算

e.g.:

一个 5×5 的 dense tensor：
$$
\begin{bmatrix} 0 & 0 & 0 & 0 & 1 \\ 0 & 2 & 0 & 0 & 0 \\ 0 & 0 & 0 & 3 & 0 \\ 0 & 0 & 0 & 0 & 0 \\ 4 & 0 & 0 & 0 & 0 \end{bmatrix}
$$
对应的 **sparse tensor** 会存成：

- **indices**（非零元素的坐标）：
  $$
  \begin{bmatrix} 0 & 1 & 2 & 4 \\ 4 & 1 & 3 & 0 \end{bmatrix}
  $$

- **values**（非零元素的值）：
  [1,2,3,4]

- **shape**：(5,5)

## Config

`mingt.utils.py`

\_\_init\_\_ with **kwargs:

```python
class CfgNode:
    """ a lightweight configuration class inspired by yacs """
    # TODO: convert to subclass from a dict like in yacs?
    # TODO: implement freezing to prevent shooting of own foot
    # TODO: additional existence/override checks when reading/writing params?

    def __init__(self, **kwargs): 
        self.__dict__.update(kwargs) # take key-value in kwargs and add them as attributes to the object
        # ==
        #for k,v in kwargs.items():
        #  setattr(self, k, v)

CfgNode(a=1, b=2)
# kwargs=={a:1, b:2}
```

`CfgNode.merge_from_args`: tree recursive get the node before the leaf, to update the value of leaf node

## parameters

```python
def __init__(self, modules: Optional[Iterable[Module]] = None) -> None:
```

- `Iterable[Module]`: it is a iterable object (e.g.: list / tuple / generator), here the type of element is `Module`
- `Optional[T]==Union[T, None]`: 该变量可以是给定类型`T`，也可以是None

## LayerNorm v.s. BatchNorm

**BatchNorm**：在 **batch 维度** 上做归一化（依赖 mini-batch 统计量）。

**LayerNorm**：在 **特征维度** 上做归一化（每个样本独立，不依赖 batch）。

```python
# layerNorm
import torch
import torch.nn as nn

ln = nn.LayerNorm(4)  # 归一化最后一维度大小为 4

x = torch.tensor([[1., 2., 3., 4.],
                  [2., 4., 6., 8.]])  # shape (2, 4)

y = ln(x)
print(y)
"""
对每个样本的最后一维进行归一化，得到的结果是：
tensor([[-1.3416, -0.4472,  0.4472,  1.3416],
        [-1.3416, -0.4472,  0.4472,  1.3416]],
"""
```

## multi-head masked self-attention layer

Attention
$$
Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V
$$


```python
class CausalSelfAttention(nn.Module):
    """
    A vanilla multi-head masked self-attention layer with a projection at the end.
    It is possible to use torch.nn.MultiheadAttention here but I am including an
    explicit implementation here to show that there is nothing too scary here.
    """

    def __init__(self, config):
        super().__init__()
        assert config.n_embd % config.n_head == 0
        # key, query, value projections for all heads, but in a batch
        self.c_attn = nn.Linear(config.n_embd, 3 * config.n_embd)
        # output projection
        self.c_proj = nn.Linear(config.n_embd, config.n_embd)
        # regularization
        self.attn_dropout = nn.Dropout(config.attn_pdrop)
        self.resid_dropout = nn.Dropout(config.resid_pdrop)
        # causal mask to ensure that attention is only applied to the left in the input sequence
        # tril = 下三角矩阵 (triangular lower)
        """
        [[1, 0, 0, 0],
         [1, 1, 0, 0],
         [1, 1, 1, 0],
         [1, 1, 1, 1]]
        """
        self.register_buffer("bias", torch.tril(torch.ones(config.block_size, config.block_size))
                                     .view(1, 1, config.block_size, config.block_size))
        self.n_head = config.n_head
        self.n_embd = config.n_embd

    def forward(self, x):
        B, T, C = x.size() # batch size, sequence length, embedding dimensionality (n_embd)

        # calculate query, key, values for all heads in batch and move head forward to be the batch dim
        q, k ,v  = self.c_attn(x).split(self.n_embd, dim=2)
        k = k.view(B, T, self.n_head, C // self.n_head).transpose(1, 2) # (B, nh, T, hs)
        q = q.view(B, T, self.n_head, C // self.n_head).transpose(1, 2) # (B, nh, T, hs)
        v = v.view(B, T, self.n_head, C // self.n_head).transpose(1, 2) # (B, nh, T, hs)

        # causal self-attention; Self-attend: (B, nh, T, hs) x (B, nh, hs, T) -> (B, nh, T, T)
        att = (q @ k.transpose(-2, -1)) * (1.0 / math.sqrt(k.size(-1)))
        att = att.masked_fill(self.bias[:,:,:T,:T] == 0, float('-inf'))
        att = F.softmax(att, dim=-1)
        att = self.attn_dropout(att)
        y = att @ v # (B, nh, T, T) x (B, nh, T, hs) -> (B, nh, T, hs)
        y = y.transpose(1, 2).contiguous().view(B, T, C) # re-assemble all head outputs side by side

        # output projection
        y = self.resid_dropout(self.c_proj(y))
        return y
```

## register_buffer()

- `register_buffer` 是 PyTorch `nn.Module` 的一个方法。
- 作用：把一个 **tensor 注册为模型的一部分**，但它不是可训练参数（不会在 `optimizer` 里更新）。
- 用途：
  - 让 buffer（比如 mask、常量）随着模型 `.to(device)`、`.cuda()` 一起移动到 GPU。
  - 保证在保存/加载模型 (`state_dict`) 时，这个 tensor 也能保存/恢复。

```python
"""
Add a buffer to the module.

        This is typically used to register a buffer that should not be
        considered a model parameter. For example, BatchNorm's ``running_mean``
        is not a parameter, but is part of the module's state. Buffers, by
        default, are persistent and will be saved alongside parameters. This
        behavior can be changed by setting :attr:`persistent` to ``False``. The
        only difference between a persistent buffer and a non-persistent buffer
        is that the latter will not be a part of this module's
        :attr:`state_dict`.

        Buffers can be accessed as attributes using given names.
"""
```

