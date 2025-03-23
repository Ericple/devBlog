+++
date = '2025-03-23T11:05:33+08:00'
draft = false
title = '将已有的Arkts项目转为Cangjie混合开发工程'
+++

> 本来没想写这一篇博客的，但我这天突然心血来潮，跟着[华子的文档](https://developer.huawei.com/consumer/cn/doc/cangjie-guides-V5/cj-cangjie-in-arkts-V5#实现仓颉互操作模块)走了一遍，发现根本跑不通，于是决定写一篇博客记录一下我的经历，让大家少走点弯路。在本文中，我将详细介绍如何将已有的Arkts项目转为Cangjie混合开发工程，包括准备工作、配置步骤和常见问题解决方法。

## 获取鸿蒙Cangjie开发插件

撰写本文时，HarmonyOS NEXT仓颉语言开发者预览版Beta招募仍在进行中，其报名周期为：2024年6月21日-2025年3月30日，后续报名开放情况需关注华为官方消息。你可以[点击这里](https://developer.huawei.com/consumer/cn/activityDetail/cangjie-beta/)查看当前是否在招募中，如果正在招募，点击“立即报名”即可开始报名。

在报名成功，并获取到预览版资格后，即可前往[下载中心](https://developer.huawei.com/consumer/cn/download/)下载`DevEco Studio-Cangjie Plugin`。对于macOS用户，如果你的浏览器在你下载完后直接自动给你解压了这个压缩包，你可能需要手动把它给压缩回去，否则DevEco Studio无法识别此插件，进而无法安装。压缩好后打开DevEco Studio，在插件页面点击从磁盘安装插件，并选择你的插件压缩包，即可安装成功。

## 添加仓颉混合库

在DevEco Studio中，右击你的项目根文件夹，选择新建-模块，再选择[Cangjie]Hybrid Static Library
![hybrid-select](img/cj-hybrid-1.png)

> 注意，我们的目标是混合开发，必须选择Hybrid，否则无法成功引用此库

## 编写Cangjie侧代码

为了更便捷地从ArkTS侧调用Cangjie函数，并降低仓颉侧的代码编写难度，我们需要导入互操作库。你可以通过以下代码引入它：

```ts
import ohos.ark_interop.*
import ohos.ark_interop_macro.Interop
```

随后，我们就可以开始编写一个简单的函数

```c
@Interop[ArkTS]
public func hello_cangjie(name: String): String {
    "Hello, ${name}!"
}
```

完整的代码长这样

```go
package ohos_app_cangjie_libxxx // libxxx应该为你自己的库名，在对照的时候无需参考

internal import cj_res_libxxx.app
import ohos.ark_interop_macro.Interop
import ohos.ark_interop.*

@Interop[ArkTS]
public func hello_cangjie(a:String):String{
    "Hello ${a}"
}
```

## 生成声明文件

编写完后，我们需要在工程目录下右击刚才创建的Cangjie Hybrid Static模块，选择`Generate Cangjie-ArkTS Interop API`，并等待执行完成。

![declare-gen](img/cj-hybrid-2.png)

完成后，`cangjie`目录下会多一个`ark_interop_api`文件夹，里面就是我们需要的声明文件

```ts
// ark_interop_api.d.ts
export declare interface CustomLib {
    hello_cangjie(a: string): string
}
```

## 导出仓颉库

在`ets`目录下新建一个`loader.ets`，并添加以下代码

```ts
import { requireCJLib } from "libark_interop_loader.so";
import { CustomLib } from "../cangjie/ark_interop_api";

let libInstance: CustomLib | undefined = undefined;

export function CJNative(): CustomLib {
  if (libInstance == undefined) {
    libInstance = requireCJLib('libohos_app_cangjie_libxxx.so') as CustomLib;
  }
  return libInstance;
}
````

这个文件中的`CJNative`函数提供了加载仓颉库的能力，用于在ArkTS侧调用仓颉函数。在ArkTS侧，你可以通过`CJNative()`函数获取到仓颉库的实例，然后调用其中的函数。

```ts
// 你的ArkTS文件
CJNative().hello_cangjie('World')
```

## 最后

至此，混合开发工程已经建立，接下来只需要将你创建的库引入到你需要的模块即可。

```json
{
  "name": "entry",
  "version": "1.0.0",
  "description": "Please describe the basic information.",
  "main": "",
  "author": "",
  "license": "",
  "dependencies": {
    "libxxx": "file:../libxxx"
  }
}
```

调用代码如下：

```ts
promptAction.showToast({ message: CJNative().hello_cangjie("Cangjie") });
```
