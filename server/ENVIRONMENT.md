[TOC]

## GCC

```bash
export CXX=/usr/bin/g++-11
export CC=/usr/bin/gcc-11
export CUDAHOSTCXX=/usr/bin/g++-11
```

将当前shell的gcc指向gcc-11

```bash
alias gcc=/usr/bin/gcc-11
alias g++=/usr/bin/g++-11
```

check

```bash
gcc --version
```

## CUDA

```bash
export CUDA_HOME=$CONDA_PREFIX
export CUDA_PATH=$CUDA_HOME
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
export PATH=$CONDA_PREFIX/bin:$PATH
```

