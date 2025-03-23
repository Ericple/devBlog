+++
date = '2025-03-23T16:09:48+08:00'
draft = false
title = 'Cangjie-ArkTS互操作的魅力之@Interop宏'
+++

> 由于目前暂时没找到Interop宏的详细文档，我只在文档中找到了Interop使用的相关线索，因此本文内容可能不完全正确。本文中出现的所有代码均为个人验证后所得的可运行代码，如有错误，烦请指正，感谢！

阅读本文可能需要一定的仓颉语言基础，可以在[这里](https://developer.huawei.com/consumer/cn/doc/cangjie-guides-V5/basic-V5)找到

如果你在阅读文档时对如何在现有ArkTS项目中新增仓颉混合开发模块或如何从ArkTS侧调用仓颉的内容有疑惑，可以参考[这篇文章](https://developer.huawei.com/consumer/cn/blog/topic/03178121339682086)

HarmonyOS上的传统Native开发采用的是C++，而随着仓颉的不断发展，使用仓颉开发高性能库也越来越接近现实，且比C++方便不少。使用`@Interop[ArkTS]`宏极大地便利了仓颉库的开发，我们只需要关心仓颉侧类、方法等的实现，而无需关心仓颉侧内容是如何挂载到ArkTS侧的。个人认为比@ohos/aki方便的多。

## Interop宏

Interop宏允许开发者将仓颉类、接口、函数快速暴露给arkTS侧使用，使用对类、接口、函数使用`@Interop[ArkTS]`标注后，再通过DevEco Studio Cangjie Plugin的`Generate Cangjie-ArkTS Interop API`功能，即可快速生成对应的.d.ts声明文件，并实现从ArkTS侧调用仓颉。

## 类的暴露

为了便于理解，先放一个基础的类的实现

```
@Interop[ArkTS]
public class JustAClass {
    var id: Int32 = 0

    public func setId(id:Int32):Unit{
        this.id=id
    }

    public func toString():String {
        "${id}"
    }

    public init(){}
}
```

这个类拥有一个私有的Int32属性，两个公共的方法和一个空的初始化方法。

使用`@Interop`宏标注的类，必须拥有一个公共初始化方法，否则会出现宏展开错误。所有私有方法或私有属性，均会在生成声明文件时被忽略，也就无法从ArkTS侧直接调用，但我们依旧可以通过公共方法访问这些私有属性。

对于上面给出的实例，`Generate Cangjie-ArkTS Interop API`生成了如下声明文件

```ts
export declare class JustAClass {
    setId(id: number): void
    toString(): string
}
```

可以看到，返回值类型为`Unit`的`setId`方法在转换后变成了无返回值(void)，而返回值类型为`String`的方法直接转换为了`string`。而私有属性`id`则没有被直接暴露到ArkTS环境中。

想要把`id`直接暴露给`ArkTS`侧也十分简单，只需要在其前面增加`public`声明即可，因此我们可以总结以下内容：

- 使用`@Interop[ArkTS]`宏对类进行标注，即可将该类暴露至`ArkTS`侧调用
- 对于要暴露至`ArkTS`侧的类，必须显式地声明一个公共初始化方法
- 所有公共方法和属性，均会被暴露至`ArkTS`侧，而私有方法及私有属性，则不会被暴露至`ArkTS`侧
- 对于无返回值方法，使用`Unit`即可
- 仓颉开发插件的`Generate Cangjie-ArkTS Interop API`功能可以自动将仓颉基本类型转换为`ArkTS`基本类型，但如果需要返回类或接口，则要返回的类/接口必须也用`@Interop[ArkTS]`进行标注

> 有关静态方法、静态属性的暴露仍在研究中，目前看来可能是无法暴露静态成员给ArkTS侧的，如果有人知道方法，欢迎在评论区交流。

## 函数的暴露

同样地，先放一个可用的实例：

```
@Interop[ArkTS]
public func hello_cangjie(a: String): String {
    "Hello ${a}"
}
```

函数的暴露比较简单，只需要保证：

- 函数必须被`public`修饰
- 所有的参数、返回值都必须是`@Interop[ArkTS]`标注过的类或基本类型

对于上面这个实例函数，在转换后的声明文件中长这样

```ts
export declare interface CustomLib {
    hello_cangjie(a: string): string
}
```

可以看到，游离的方法被包裹在了一个叫CustomLib的接口中。如果你同时暴露了一个类，则这个类也会被暴露在CustomLib中。被包裹在这里面的函数或类要如何调用，可以参考[这篇文章](https://developer.huawei.com/consumer/cn/blog/topic/03178121339682086)
