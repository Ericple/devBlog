+++
date = '2025-02-09T23:04:19+08:00'
draft = false
title = 'OpenHarmony application_systemui编译过程'
+++

> 本文将分享在Ubuntu 20.04上编译`OpenHarmony`系统应用`systemui`的过程
> 由`梨猫先生`原创，首发于[肥肥梨猫的小窝](https://peercat.cn/)，转载请注明出处
## 源码准备

{{< alert >}}
如果你不打算使用repo下载源码，而是从镜像站下载后解压，请跳转到[编译准备](#编译准备)
{{< /alert >}}
<!--more-->


在此部分，我将引导你建立`OpenHarmony`的编译环境，相关参考请前往[子系统编译资料](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-all.md)查看。

初始路径：`~/`

### 1.安装git/curl/python3.8/python3-pip

```bash
sudo apt-get install git curl python3.8 python3-pip -y
```

### 2.安装repo工具

将`repo`下载到`~/bin`下
```bash
sudo curl -s https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > usr/bin/repo
```
给所有用户的`repo`都加上执行权限
```bash
sudo chmod a+x usr/bin/repo
```
配置repo地址
```bash
pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests
```

### 3.配置git

```bash
git config --global user.name "yourname"
git config --global user.email "your-email-address"
git config --global credential.helper store
```

### 4.获取源码

为源码准备文件夹
```bash
mkdir ohos
```
进入文件夹
```bash
cd ohos
```
初始化repo仓库
```bash
repo init -u https://gitee.com/ohos_port/manifest -b fajita-OpenHarmony-3.2-Release --no-repo-verify
```
拉取代码
```bash
repo sync -c && repo forall -c 'git lfs pull'
```

## 编译准备

初始路径：`xxx/你的源码目录/`

### 1.执行prebuilts_download

```bash
./build/prebuilts_download.sh
```

### 2.准备环境

```bash
./build/build_scripts/env_setup.sh
```

### 3.编译systemui

```bash
./applications/standard/hap/build.sh --project=~/[源码目录]/applications/standard/systemui --build_sdk=true --npm=@ohos:https://repo.harmonyos.com/npm
```

第一次进行编译时将编译SDK，请耐心等待。