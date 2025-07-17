

### CUDA体系

nvidia中cuda驱动版本

`nvidia-smi`

conda中的nvcc

`nvcc --version`

`which nvcc`

`/usr/local`下可查看服务器安装的cuda版本

切换cuda版本：

```
vim ~/.bashrc
source  ~/.bashrc
```

**CUDA** (Compute Unified Device Architecture): GPU和程序（如python）之间的桥梁。NVIDIIA推出的通用并行计算架构，该架构使GPU能解决复杂的计算问题。包含CUDA指令集架构(ISA)以及GPU内部的并行计算引擎。开发人员可以使用C语言为CUDA架构编写程序，所编写出的程序可以在支持CUDA的处理器上以超高性能运行。

=> CUDA是让python等程序语言可以同时在CPU和GPU上跑的平台。

![图一](https://i-blog.csdnimg.cn/blog_migrate/37444668e40597a17991da9029cafb5a.png)

- **CUDA Toolkit** (nvidia) [Container]: CUDA完整的工具安装包，其中提供了Nvidia驱动程序、开发CUDA程序相关的开发工具包等可供安装等选项，包括CUDA程序等编译器、IDE、调试器等，CUDA程序所对应的各式库文件以及它们的头文件。
- **CUDA Driver** [OS层]:运行CUDA应用程序需要系统至少有一个具有CUDA功能的GPU和CUDA Tookit兼容的驱动程序。CUDA Drivier向后兼容

#### CUDA结构

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e772b4ffffe3adbf339fb88c0b942fa.png)

- **CUDA Tookit**：包含CUDA Libraries和runtime
  - CUDA Toolkit (nvidia)： CUDA完整的工具安装包，其中提供了 Nvidia 驱动程序、开发 CUDA 程序相关的开发工具包等可供安装的选项。包括 CUDA 程序的编译器、IDE、调试器等，CUDA 程序所对应的各式库文件以及它们的头文件。
  - CUDA Toolkit (Pytorch)： CUDA不完整的工具安装包，其主要包含在使用 CUDA 相关的功能时所依赖的动态链接库。不会安装驱动程序，也不会安装编译工具(nvcc)。pytorch带的CUDA不会安装runtime层和以下的层，包括nvcc和CUDA driver。
  
- **NVCC**：CUDA的编译器，属于runtime层
- **cuDNN** (NVIDIA CUDA Deep Neural Network library)，是专门针对深度神经网络中的基础操作而设计基于GPU的加速库。



Docker v.s. 虚拟环境

- **虚拟环境**：隔离Python程序的依赖项，即在一个虚拟环境中，包含了特定版本的Python解释器和Python库，当激活该虚拟环境时，会屏蔽掉虚拟环境以外的python解释器和python库。
- **Docker**：隔离整个系统，更接近虚拟机，同时docker可以有不同层次的封装