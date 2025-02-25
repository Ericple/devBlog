+++
date = '2025-02-13T15:01:30+08:00'
draft = false
title = '实现追加器'
tags = ['新手', '文档']
+++

所有追加器都继承自同一抽象类`AbstractAppender`，开发者可以自行实现其中关键方法，并通过`Logger`中的`addAppender`方法将追加器绑定至`Logger`。

## 追加器的生命周期

追加器有以下周期

`onLog(level, tag, time, count, message)`

- `level` Level - 日志等级
- `tag` string - 日志所属类名
- `time` number - 日志打印时间戳
- `count` number - 该日志为此`Appender`处理的第`count`条日志
- `message` string - 日志内容

当有日志被记录时，追加器的onLog方法会被Logger调用，开发者可自行实现日志功能。

`onTerminate()`

当追加器被终止时，onTerminate方法会被Logger调用，开发者应当在此处回收不再被使用的内存，或取消不必要的事件订阅。

## 实例

这个例子实现了一个最简单的追加器，用于将日志输出到控制台上

```ts
class SimpleAppenderClass extends AbstractAppender {
    onLog(level: Level, tag: string, time: number, count: number, message: string): void {
        console.log(message);
    }
    onTerminate(): void {
        return;
    }
}
export const SimpleAppender = new SimpleAppenderClass;
```

```ts
import { SimpleAppender } from './file.ets';
import { Logger, LogManager } from '@pie/log4a';

class SomeClass {
    logger: Logger = LogManager.getLogger(this).addAppender(SimpleAppender);
}
```

## 尝试一下

<script setup>
import {defineAsyncComponent} from 'vue';
import {inBrowser} from 'vitepress';

const DemoEditor = inBrowser ? defineAsyncComponent(()=>import('../components/DemoEditor.vue')):()=>null;
</script>

<DemoEditor code="class SimpleAppenderClass extends AbstractAppender {
    onLog(level, tag, time, count, message) {
        console.log(message);
    }
    onTerminate() {
        return;
    }
}
const SimpleAppender = new SimpleAppenderClass;
class SomeClass {
    logger = LogManager.getLogger(this).addAppender(SimpleAppender);
    hello(){
        this.logger.info('Hello, World!')
    }
}
const someClassInstance = new SomeClass;
someClassInstance.hello();" />