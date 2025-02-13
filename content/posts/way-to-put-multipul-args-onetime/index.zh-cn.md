+++
date = '2025-02-09T22:54:07+08:00'
draft = false
title = '在开发中让一个参数可以传递多个信息的技巧'
+++
## 起因

在鸿蒙开发中，我发现fs.open及其它个别文件操作函数中可以传入一个很有意思的值：fs.OpenMode。它的特点在于可以通过一个参数来传递多个信息：

```ts
fs.openSync('path', fs.OpenMode.CREATE | fs.OpenMode.APPEND);
```

通过代码提示，可以看到`fs.OpenMode`其实是一组八进制数字

```ts
namespace OpenMode {
        const READ_ONLY = 0o0;
        const WRITE_ONLY = 0o1;
        const READ_WRITE = 0o2;
        const CREATE = 0o100;
        const TRUNC = 0o1000;
        const APPEND = 0o2000;
        const NONBLOCK = 0o4000;
        const DIR = 0o200000;
        const SYNC = 0o4010000;
}
```

## 原理

不难发现，这是通过位运算来实现的，**在参数中进行按位或运算，在函数内部通过按位与运算即可解出参数中传递的信息。**

### 按位或

二进制数据中只有0和1，按位或是指将两个二进制数据右端对齐，左侧按需补0对齐后，根据“有1则1，无1则0”的法则进行运算得到结果，下面是一个示意图：

![按位或](img/1695818536.png)

按位或的运算符号是 `|`，在代码中可以通过以下方式表示：

```ts
const a = 2;
const b = 4;
const c = a | b;
```

### 按位与

和按位或类似，对齐两个二进制数据后，按位与的运算法则是“都1才1，有0则0”，下面是按位与的运算示意图

![按位与](img/3517941759.png)

按位与的运算符号是 `&`，在代码中可以通过以下方式表示：

```ts
const a = 2;
const b = 4;
const c = a & b;
```

## 实践

了解了原理后，我们可以编写一个运用或与运算来解析参数的案例：

```ts
enum ExampleFunctionParams {
    SAY_HELLO = 2,
    SAY_GOODBYE = 64
}
function exampleFunction(param:number): void {
    console.log('function start');
    // 如果参数中含有SAY_HELLO
    if(param & ExampleFunctionParams.SAY_HELLO) {
        console.log('Hello world!');
    }
    // 如果参数中含有SAY_GOODBYE
    if(param & ExampleFunctionParams.SAY_GOODBYE) {
        console.log('Goodbye world!');
    }
}
exampleFunction(ExampleFunctionParams.SAY_HELLO);
exampleFunction(ExampleFunctionParams.SAY_GOODBYE);
exampleFunction(ExampleFunctionParams.SAY_HELLO | ExampleFunctionParams.SAY_GOODBYE);
```

运行结果：

```bash
function start
Hello world!
function start
Goodbye world!
function start
Hello world!
Goodbye world!
```