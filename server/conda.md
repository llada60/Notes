[TOC]

# Simple command

```conda create -n xxx python=3.x ```： 安装新的conda环境

```conda create -n xxx --clone AAA```：复制conda环境

```conda activate xxx```：启动环境

```conda env list```

```conda remove -n xxx --all```

```conda deactivate```：退出当前环境

# conda-pack package

## Use Cases

- Bundling an application with its environment for deployment
- Packaging a conda environment for use with Apache Spark when deploying on YARN ([see here](https://conda.github.io/conda-pack/spark.html) for more information).
- Packaging a conda environment for deployment on Apache YARN. One way to do this is to use [Skein](https://jcrist.github.io/skein/).
- Archiving an environment in a functioning state. Note that a more sustainable way to do this is to specify your environment as a [environment.yml](https://conda.io/docs/user-guide/tasks/manage-environments.html#sharing-an-environment), and recreate the environment when needed.
- Packaging an environment as single executable with entrypoint to run on execution (see the instructions for [Linux and macOS](https://conda.github.io/conda-pack/unix-binary.html)).
- *BETA*: Packaging a conda environment as a standard Cloudera parcel. This is a newly added capability. It has been tested on a live cluster, but different cluster configurations may produce different results. We welcome users to [file an issue](https://github.com/conda/conda-pack/issues) if necessary. [See here](https://conda.github.io/conda-pack/parcel.html) for more information).

## Commandline Usage

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

## API Usage

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
