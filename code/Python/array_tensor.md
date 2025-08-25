[TOC]

# Tensor / array change shape

## 基础变形

**`view()`** (pytorch)

只能对 **内存是连续（contiguous）** 的 tensor 使用。

如果 tensor 不是连续的（比如经过 `transpose()`、`permute()` 之类操作），你必须先调用 `.contiguous()` 才能用 `view()`。

**`reshape()`** (PyTorch / NumPy)

更灵活，自动处理非连续内存；可能共享，也可能复制。

**`resize_()`** (PyTorch, inplace)

原地改变大小，可能导致数据被覆盖（不常用，危险）。

## 维度调整

**`squeeze()`**

- 去掉 size=1 的维度。

```
x = torch.randn(1, 3, 1, 5)
x.squeeze()       # (3, 5)
x.squeeze(0)      # (3, 1, 5)  只去掉dim=0
```

**`unsqueeze()`**

- 增加一个维度。

```
x = torch.randn(3, 5)
x.unsqueeze(0)    # (1, 3, 5)
x.unsqueeze(2)    # (3, 5, 1)
```

**`expand()`**

- 在不复制数据的情况下扩展维度（只能扩展 1 → n）。

```
x = torch.randn(1, 3)
x.expand(4, 3)    # (4, 3)
```

**`repeat()`**

- 显式复制数据，生成新的张量。

```
x = torch.tensor([1, 2])
x.repeat(3)       # tensor([1, 2, 1, 2, 1, 2])
```

## 维度交换 / 转置

**`transpose()`**

- 交换两个维度。

```
x = torch.randn(2, 3, 4)
x.transpose(1, 2).shape   # (2, 4, 3)
```

**`permute()`**

- 任意顺序重排维度。

```
x = torch.randn(2, 3, 4)
x.permute(2, 0, 1).shape  # (4, 2, 3)
```

**`T` 属性**

- 矩阵转置（二维情况的简写）。

```
x = torch.randn(3, 4)
x.T.shape    # (4, 3)
```

## 展平 / 拼接

**`flatten()`**

- 展平为一维，或部分展平。

```
x = torch.randn(2, 3, 4)
x.flatten()        # (24,)
x.flatten(1)       # (2, 12)  # 从dim=1开始展平
```

**`cat()`**

- 按维度拼接张量。

```
torch.cat([a, b], dim=0)
```

**`stack()`**

- 新增一个维度后拼接。

```
torch.stack([a, b], dim=0)
```

## NumPy 特有

**`np.newaxis`**

- 插入新维度（等价于 `unsqueeze`）。

```
x = np.array([1, 2, 3])
x[:, np.newaxis].shape   # (3, 1)
```

**`np.ravel()`**

- 拉平，返回尽可能共享内存的 view。

**`np.transpose()` / `.T`**

- 维度置换。

**`np.expand_dims()` / `np.squeeze()`**

- 功能同 PyTorch。

