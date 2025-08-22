[TOC]

## Basic usage

**Create and manage environments**:

```conda create -n myenv python=3.x ```： 安装新的conda环境

```conda create -n myenv --clone oldenv```：复制conda环境

```conda activate xxx```：启动环境

```conda env list```

```conda remove -n xxx --all```

```conda deactivate```：退出当前环境

**Install packages**:

```bash
conda install numpy pandas            # Basic packages
conda install pytorch -c pytorch      # From specific channel
pip install package-name              # For packages not in conda
```

### Cuda Version Control

```bash
conda install pytorch torchvision torchaudio pytorch-cuda=12.1
conda install \
    nvidia/label/cuda-12.1.0::cuda-toolkit \
    libcusparse-dev \
    libcusolver-dev \
    --override-channels \
    -c nvidia/label/cuda-12.1.0
```

Args:

- `-c, --channel`

  指定额外的包搜索通道。这些通道按给定顺序搜索，可以是 URL、本地目录（使用 'file://' 语法）或简单路径（如 '/home/conda/mychan' 或 '../mychan'）。然后，搜索默认或 .condarc 中的通道（除非使用了 --override-channels）。可以使用 'defaults' 来获取 conda 的默认包。还可以使用任何名称，.condarc 的 channel_alias 值将被预先添加。默认的 channel_alias 是 https://conda.anaconda.org/。

- `--use-local`

  使用本地构建的包。等同于 '-c local'。

- `--override-channels`

  完全忽略配置文件中的默认channels，仅在-c指定的channel中搜索包

## GCC

```bash
conda install -c conda-forge gcc=11 gxx=11
```



## Environment

```bash
export CUDA_HOME=$CONDA_PREFIX
export CUDA_PATH=$CUDA_HOME
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
export PATH=$CONDA_PREFIX/bin:$PATH

export CC=$CONDA_PREFIX/bin/gcc
export CXX=$CONDA_PREFIX/bin/g++
export CFLAGS="-I$CONDA_PREFIX/include"

export CXXFLAGS="-I$CONDA_PREFIX/include"
export LDFLAGS="-L$CONDA_PREFIX/lib"
```

```
echo $CUDA_HOME
echo $CUDA_PATH
echo $LD_LIBRARY_PATH
echo $PATH
echo $CC
echo $CXX
echo $CFLAGS
```



## conda install [pitfalls]

Checking torch's cuda version

```bash
python -c "import torch; print('Torch:', torch.__version__, '| CUDA available:', torch.cuda.is_available(), '| torch.version.cuda:', torch.version.cuda)"
```

## Environment Sharing

Export your environment:

```bash
conda env export > environment.yml            # Full export
conda env export --from-history > minimal.yml # Basic dependencies
```

Create from file:

```bash
conda env create -f environment.yml           # Create environment from file
```



## MiniConda Install

Conda is our recommended tool for managing Python environments. It provides:

- Isolated environments for different projects
- Comprehensive package management
- Import and export environment easily

Install Miniconda:

```bash
# Download and install
mkdir -p ~/miniconda3
wget <https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh> -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```

Exit your session as suggested by the installer, and re-log in :

```powershell
# Load the conda binaries
source ~/.bashrc

# Initialize conda environment
conda init
```

## conda-pack package [no working]

### Use Cases

- Bundling an application with its environment for deployment
- Packaging a conda environment for use with Apache Spark when deploying on YARN ([see here](https://conda.github.io/conda-pack/spark.html) for more information).
- Packaging a conda environment for deployment on Apache YARN. One way to do this is to use [Skein](https://jcrist.github.io/skein/).
- Archiving an environment in a functioning state. Note that a more sustainable way to do this is to specify your environment as a [environment.yml](https://conda.io/docs/user-guide/tasks/manage-environments.html#sharing-an-environment), and recreate the environment when needed.
- Packaging an environment as single executable with entrypoint to run on execution (see the instructions for [Linux and macOS](https://conda.github.io/conda-pack/unix-binary.html)).
- *BETA*: Packaging a conda environment as a standard Cloudera parcel. This is a newly added capability. It has been tested on a live cluster, but different cluster configurations may produce different results. We welcome users to [file an issue](https://github.com/conda/conda-pack/issues) if necessary. [See here](https://conda.github.io/conda-pack/parcel.html) for more information).

### Commandline Usage

`conda-pack` is primarily a commandline tool. Full CLI docs can be found [here](https://conda.github.io/conda-pack/cli.html).

One common use case is packing an environment on one machine to distribute to other machines that may not have conda/python installed.

On the **source machine**

```
# Pack environment my_env into my_env.tar.gz
$ conda pack -n my_env

# Pack environment my_env into out_name.tar.gz
$ conda pack -n my_env -o out_name.tar.gz

# Pack environment located at an explicit path into my_env.tar.gz
$ conda pack -p /explicit/path/to/my_env
```

On the **target machine**

```
# Unpack environment into directory `my_env`
$ mkdir -p my_env
$ tar -xzf out_name.tar.gz -C my_env

# Use python without activating or fixing the prefixes. Most python
# libraries will work fine, but things that require prefix cleanups
# will fail.
$ ./my_env/bin/python

# Activate the environment. This adds `my_env/bin` to your path
$ source my_env/bin/activate

# Run python from in the environment
(my_env) $ python

# Cleanup prefixes from in the active environment.
# Note that this command can also be run without activating the environment
# as long as some version of python is already installed on the machine.
(my_env) $ conda-unpack

# At this point the environment is exactly as if you installed it here
# using conda directly. All scripts should work fine.
(my_env) $ ipython --version

# Deactivate the environment to remove it from your path
(my_env) $ source my_env/bin/deactivate
```

### API Usage

`conda-pack` also provides a Python API, the full documentation of which can be found [here](https://conda.github.io/conda-pack/api.html). The API mostly mirrors that of the `conda pack` commandline. Repeating the examples from above:

```
import conda_pack

# Pack environment my_env into my_env.tar.gz
conda_pack.pack(name="my_env")

# Pack environment my_env into out_name.tar.gz
conda_pack.pack(name="my_env", output="out_name.tar.gz")

# Pack environment located at an explicit path into my_env.tar.gz
conda_pack.pack(prefix="/explicit/path/to/my_env")
```
