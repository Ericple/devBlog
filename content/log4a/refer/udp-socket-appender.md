+++
date = '2025-02-13T15:01:30+08:00'
draft = false
title = 'UDPAppender'
tags = ['API参考']
+++

> 提供通过UDP协议连接服务器并将日志实时推送到服务端的能力，需要开发者自行申请`ohos.permission.INTERNET`权限

## `constructor(config)`

- `config` UDPSocketAppenderOptions
  - `address` string - 服务器地址
  - `port` number - 服务器端口号

新建一个`UDPSocketAppender`

## `onLog(level, message)`

- `level` Level - 日志等级
- `message` string - 日志内容

当被绑定的宿主Logger记录日志时会调用此方法

## `onTerminate()`

终止此Appender的所有日志记录活动