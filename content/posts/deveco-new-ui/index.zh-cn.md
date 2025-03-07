+++
date = '2025-03-07T11:18:18+08:00'
draft = false
title = 'DevEco Studio开启New UI'
series = ["OpenHarmony"]
series_order = 1
tags = ["美化"]
+++

本文记录了开启DevEco Studio中New UI的途径，并通过界面编辑解决了开启新UI后部分组件不可用的问题

> 本文快捷键以macOS为例，各平台的快捷键可能有所不同，请操作时注意区别

## 开启New UI

- 按CommandOrControl+Shift+A打开操作搜索弹窗，搜索"registry"

![deveco-csa](img/deveco-csa-popup.png)

> 该步骤的快捷方式在macOS 15及以上可能存在与系统快捷方式冲突的问题，对于此类情况，你可以在首选项-快捷键中对DevEco Studio快捷键进行编辑。
> 如果你不想更改快捷键，也可以通过菜单中的"帮助-查找操作"打开同样的搜索弹窗

- 单击搜索结果中的"注册表"
- 找到"ide.experimental.ui"，并勾选此项

![check-experimental-ui](img/check-experimental-ui.png)

- 点击"确认"并重启IDE

## 恢复顶栏工具配置

开启新UI后，顶栏右侧会缺失部分工具，严重影响开发效率，我们可以通过修改外观来恢复这些工具。

- 按下"CommandOrControl+,"打开首选项

![devecp-setting-index](img/deveco-setting-index.png)

- 展开左侧列表中的"外观和行为“，选择"菜单和工具栏"
- 在右侧按顺序展开"主工具栏"-"Right"

![deveco-setting-ap-right](img/deveco-setting-ap-right.png)

- 按照图示顺序添加工具即可