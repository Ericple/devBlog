+++
date = '2025-02-13T15:01:30+08:00'
draft = false
title = 'AbstractAppender'
tags = ['API参考']
+++

AbstractAppender是所有Appender的基类，所有列出的方法均可在其他Appender中调用

## `getName()`

获取Appender名称

## `getCurrentHistory()`

获取当前会话产生的所有日志

## `setLayout(layout)`  <Badge type="tip" text="1.4.0 +" />

- `layout` T extends AbstractLayout - 日志布局

设置该Appender的日志布局

## `getType()`

获取当前Appender类型

## `onTerminate()`

终止当前Appender的输出，并清理内存垃圾