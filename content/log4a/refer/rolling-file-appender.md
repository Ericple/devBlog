+++
date = '2025-02-13T15:01:30+08:00'
draft = false
title = 'RollingFileAppender'
tags = ['API参考']
+++

> 提供滚动日志输出文件的能力

## `constructor(path, name, level, maxFileSize, maxCacheCount, useWorker)`

- `path` string - 日志文件路径或名称
- `name` string - 此文件追加器名称
- `level` Level - 此追加器对应日志等级
- `maxFileSize` number? - 最大日志文件大小
- `maxCacheCount` number? - 最大日志文件数量
- `useWorker` boolean? - 是否开启多线程

新建一个`RollingFileAppender`

## `matchOptions(options)`

- `options` FileAppenderOptions - 要对比的追加器配置

判断当前追加器配置和给出配置是否完全吻合

## `onLog(level, message)`

- `level` Level - 日志等级
- `message` string - 日志内容

当被绑定的宿主Logger记录日志时会调用此方法

## `onTerminate()`

终止此Appender的所有日志记录活动

## `getAllHistory()`

获取包含缓存的所有历史日志

## `clearCurrentHistory()`

删除所有日志缓存