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

### 编译

为了避免污染，我们新建一个文件夹用于存放编译配置

```bash
mkdir build_ohos
cd build_ohos
```

我们可以使用SDK包含的cmake来生成交叉编译的配置文件

假设你的SDK路径为`/Applications/DevEco-Studio.app/Contents/sdk/default/openharmony`，则我们可以直接运行以下命令

```bash
/Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/native/build-tools/cmake/bin/cmake -DCMAKE_TOOLCHAIN_FILE=/Applications/DevEco-Studio.app/Contents/sdk/default/openharmony/native/build/cmake/ohos.toolchain.cmake -DOHOS_ARCH=arm64-v8a .. -L
```

这条命令使用了SDK内置的cmake和ohos.toolchain.cmake配置，生成对应的配置文件。生成完毕后，我们在`build_ohos`目录下直接运行`make`命令即可开始构建。

![fftw-compile](img/fftw-compile-result.png)

## 测试

我们新建一个支持native的工程，将构建出来的三个.so产物放到工程下的libs文件夹内，然后从`fftw3`目录下找到`fftw3.h`头文件并放入`src/main/cpp`下。

![fftw-pl](img/fftw-project-libs.png) ![fftw-ph](img/fftw-project-headers.png)

编写一个测试函数，测试我们的编译产物是否有效

> 这里我借用了aki的语法糖简化代码，如果你看不太懂，可以参考[@ohos/aki](https://gitcode.com/openharmony-sig/aki)

```c
#include <aki/jsbind.h>
#include <fftw3.h>
#include <cmath>
#define LOG_DOMAIN 0xF010
#define LOG_TAG "MUSIC_SHEET"
#include <hilog/log.h>

auto fftw_test()->bool {
    int i;
    fftw_complex *din, *out;
    fftw_plan p;
    din=(fftw_complex*)fftw_malloc(sizeof(fftw_complex)*5);
    out=(fftw_complex*)fftw_malloc(sizeof(fftw_complex)*5);
    if((din==NULL)||(out==NULL))return false;
    for(i=0;i<5;i++){
        din[i][0]=i+1;
        din[i][1]=0;
    }
    p=fftw_plan_dft_1d(5,din,out,FFTW_FORWARD,FFTW_ESTIMATE);
    fftw_execute(p);
    fftw_destroy_plan(p);
    fftw_cleanup();
    for(i=0;i<5;i++){
        OH_LOG_DEBUG(LogType::LOG_APP,"fftw native input %{public}f %{public}f",out[i][0], out[i][1]);
    }
    for(i=0;i<5;i++){
        OH_LOG_DEBUG(LogType::LOG_APP,"fftw native output %{public}f %{public}f",out[i][0], out[i][1]);
    }
    if(din!=NULL)fftw_free(din);
    if(out!=NULL)fftw_free(out);
    return true;
}

JSBIND_ADDON(libfft);

JSBIND_GLOBAL() {
    JSBIND_FUNCTION(fftw_test);
}
```

## 结果

查看输出结果一切正常，至此交叉编译移植fftw库已完成

![fftw-result](img/fftw-result-log.png)
