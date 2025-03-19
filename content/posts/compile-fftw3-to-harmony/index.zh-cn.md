+++
date = '2025-03-19T11:18:18+08:00'
draft = false
title = '将FFTW3交叉编译至HarmonyOS NEXT'
series = ["OpenHarmony"]
+++

## 前言

最近在研究快速傅立叶变换算法，希望在鸿蒙上可以写一些信号处理相关的代码。
查了不少资料，看了不少傅立叶变换的理论解释，也看了不少示例代码，
但最终写出来的算法运行效果总是不尽如人意，有这或那的问题，因此我最终放弃了自己写傅立叶变换算法的想法。
找到了[FFTW](https://fftw.org/index.html)库。

FFTW由麻省理工的[Matteo Frigo](https://www.fftw.org/~athena/)和[Steven G.Johnson](https://math.mit.edu/~stevenj/)两位大佬开发，
对这二位感兴趣的同学可以点击他们的名字前往主页了解更多。

不过有一点必须指出，FFTW采用的是GPL开源协议，这意味着如果你使用FFTW编写并发布软件，它必须是开源且免费的，我编译到HarmonyOS NEXT只是为了试水，
并且它并不会随我的任何软件(闭源)发行。

## 编译准备

> 我的编译环境是macOS Sequoia 15.3.2，已经安装好brew，使用zsh作为终端。

### 源码获取

首先需要获取fftw的源码。虽然你在FFTW官网上可以找到GitHub入口，但实际上你无法通过GitHub克隆下来的代码编译出fftw本身。它的代码仓库包含的是代码生成器，
而必须使用特定的软件来调用这个生成器才能生成可供编译的源代码。因此我们只能在官网的Introduction中找到[下载页面](https://fftw.org/download.html)，
并直接下载FFTW的源码tar包，我这里下载的是fftw-3.3.10.tar.gz，下载后解压到你想要的位置即可。

![download_page](img/fftw_download_page.png)

### 安装SDK

为了交叉编译，我们需要下载HarmonyOS NEXT SDK，相信能跟着这篇文章进行交叉编译的同学，应该已经装好了。如果你没有单独安装过，但是你安装了DevEco Studio，
那么你可以在你的`/Applications/DevEco-Studio.app/Contents/sdk/default/openharmony`下找到你的sdk。

如果你确实没有安装，你可以去[这里](https://developer.huawei.com/consumer/cn/download/)下载最新的IDE，里面包含了最新的SDK。

### 准备toolchain.cmake

为了让编译工具知道我们要交叉编译，我们需要告诉编译工具我们使用的编译工具链、编译目标系统、目标架构等信息。

在fftw源码路径下新建一个`toolchain.cmake`文件，往里面添加这些内容

```cmake
# 设置目标系统
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# 设置系统根目录，里面放着依赖库
set(CMAKE_SYSROOT /Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/native/sysroot)
set(CMAKE_FIND_ROOT_PATH ${CMAKE_SYSROOT})

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)  # 不在 sysroot 中查找程序
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)   # 只在 sysroot 中查找库
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)   # 只在 sysroot 中查找头文件

# 告诉cmake我们要使用harmony os的llvm工具链进行编译，这里根据你的实际位置替换路径
set(CMAKE_C_COMPILER /Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/native/llvm/bin/clang)
set(CMAKE_CXX_COMPILER /Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/native/llvm/bin/clang++)

set(CMAKE_C_FLAGS "--target=arm-linux-ohosmusl")
set(CMAKE_CXX_FLAGS "--target=arm-linux-ohosmusl")
```

保存后，即可准备进行编译。

## 编译

编译的过程就很简单了，首先在fftw源码路径下新建一个文件夹，用于存放你的预构建产物，然后进入这个文件夹

```bash
mkdir build
cd build
```

然后调用cmake，别忘了我们刚才写的`toolchain.cmake`，在这时候要用到

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake ..
```

等待cmake运行完成，预构建产物生成完毕，即可开make，make完后在你新建的文件夹下就可以看到
`libfftw3.so`/`libfftw3.so.3`/`libfftw3.so.3.6.9`，中间那个就是本体。

> 未完待续
