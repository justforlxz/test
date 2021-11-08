---
title: 在DeepinLinux下使用nVidia CUDA
date: 2018-06-28 03:21:50
tags:
  - Linux
  - nVidia Cuda
---

CUDA（Compute Unified Device Architecture，统一计算架构）是由NVIDIA所推出的一种集成技术，是该公司对于GPGPU的正式名称。通过这个技术，用户可利用NVIDIA的GeForce 8以后的GPU和较新的Quadro GPU进行计算。亦是首次可以利用GPU作为C-编译器的开发环境。NVIDIA营销的时候，往往将编译器与架构混合推广，造成混乱。实际上，CUDA可以兼容OpenCL或者自家的C-编译器。无论是CUDA C-语言或是OpenCL，指令最终都会被驱动程序转换成PTX代码，交由显示核心计算。

在论坛上看到有些用户希望在deepin下使用CUDA，但是他们采取的做法往往是手动下载nvidia的二进制文件，直接进行安装。

但是这样会破坏一部分的glx链接，导致卸载的时候无法彻底恢复，结果就是系统因为卸载nvidia驱动而废掉，所以我推荐使用包管理器的方式安装nvidia驱动和cuda相关的东西，尽量不要手动修改。

<!-- more -->

需要安装的很少，只有五个包，不过会依赖很多nvidia的库，总量还是有一些的。

```
sudo apt install nvidia-cuda-toolkit nvidia-profiler nvidia-visual-profiler nvidia-cuda-doc nvidia-cuda-dev
```

nvcc是cuda的编译器，它目前只支持g++5，所以还需要安装g++5。

```
sudo apt install g++-5
```

然后，重启一下计算机。

[这里有个小栗子，可以用来测试cuda是否能够成功编译和运行](https://bingliu221.gitbooks.io/learn-cuda-the-simple-way/content/chapter2.html/)

**将以下代码保存为 main.cu**

```
#include <stdio.h>

__global__ void vector_add(const int *a, const int *b, int *c) {
    *c = *a + *b;
}

int main(void) {
    const int a = 2, b = 5;
    int c = 0;

    int *dev_a, *dev_b, *dev_c;

    cudaMalloc((void **)&dev_a, sizeof(int));
    cudaMalloc((void **)&dev_b, sizeof(int));
    cudaMalloc((void **)&dev_c, sizeof(int));

    cudaMemcpy(dev_a, &a, sizeof(int), cudaMemcpyHostToDevice);
    cudaMemcpy(dev_b, &b, sizeof(int), cudaMemcpyHostToDevice);

    vector_add<<<1, 1>>>(dev_a, dev_b, dev_c);

    cudaMemcpy(&c, dev_c, sizeof(int), cudaMemcpyDeviceToHost);

    printf("%d + %d = %d, Is that right?\n", a, b, c);

    cudaFree(dev_a);
    cudaFree(dev_b);
    cudaFree(dev_c);

    return 0;
}
```

编译:

```
nvcc main.cu
```

运行:

```
./a.out
```

如果一切顺利，在编译的时候就不会有报错，不过在我的环境下nvcc会有架构被弃用的警告，本着只要不error就算没事的原则，我们无视这条警告即可。

输出结果:

```
2 + 5 = 0, Is that right?
```
