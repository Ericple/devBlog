+++
date = '2025-02-09T22:59:56+08:00'
draft = false
title = '一次TaskPool报错10200006的排故历程'
+++

## 问题

今日上午，在探究开源鸿蒙任务池的过程中，发现自己写的函数无论如何都没有运行效果，故完善了所有的错误捕获。发现出现了以下报错

<!--more-->

```json
{"code":10200006}
```

前往开源鸿蒙文档查询，发现该错误意味者参数不可解析。

此时代码如下：

```Main.ets
import TaskPool from '@ohos.taskpool';
import { SimpleThread } from './Target';


export async function SimpleThread_run(self: SimpleThread): Promise<SimpleThread> {
  let t = new TaskPool.Task(SimpleThread_add, self);
  return new Promise<SimpleThread>((resolve, reject) => {
    TaskPool.execute(t)
      .then((result: SimpleThread) => {
        resolve(result);
      })
      .catch((e: Object) => {
        reject(e);
      })
  });
}

@Concurrent
export function SimpleThread_add(self: SimpleThread): SimpleThread {
  try {
    self.val++;
    console.log(`UserLog:${self.val}`);
    return self;
  } catch (e) {
    console.log(`UserLog:${JSON.stringify(e)}`);
    return self;
  }
}

```

UIAbility中使用run函数的代码如下：

```
import { SimpleThread_run } from '../../java/Main';
import { SimpleThread } from '../../java/Target';

@Entry
@Component
struct Index {
  @State message: SimpleThread = new SimpleThread();

  build() {
    Row() {
      Column() {
        Text(`${this.message.val}`)
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
      }
      .width('100%')
    }
    .height('100%')
    .onClick(() => {
      this.message.val++;
      SimpleThread_run(this.message) // <=========使用的地方
        .then(newMessage => {
          console.log(`UserLog:${JSON.stringify(newMessage)}`)
          this.message = newMessage;
        })
        .catch((e: Object) => {
          console.log(`UserLog:${JSON.stringify(e)}`)
        })
    })
  }
}

```

发现，@State装饰的变量不可以用于入参@Concurrent装饰的函数，将message的装饰器去除即可。

那么是不是所有被装饰的均不能入参呢？

尝试了在子组件中使用@Prop装饰message变量，同样出现10200006错误。

## 原因

因此大致可以推断:
> 使用装饰器修饰的变量不能作为@Concurrent装饰的函数的参数。

## 解决方案

此外，还有一个解决方法：
> 对装饰器修饰的变量进行深拷贝，再作为参数传入。