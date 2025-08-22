[TOC]

## GPU Computing

> CUDA is only available on compute nodes, not on the gateway. Check available versions on compute nodes:
>
> ```
> ls -ld /usr/local/cuda*
> ```

### **CUDA Configuration**

Set CUDA version in your `.bashrc` or interactive session:

```bash
# Example for CUDA 12.5
export CUDA_HOME=/usr/local/cuda-12.5 
export PATH=$CUDA_HOME/bin:$PATH:/bin:/usr/bin 
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

Verify setup:

```bash
nvcc --version              # Check CUDA compiler
nvidia-smi                  # Check GPU status
```