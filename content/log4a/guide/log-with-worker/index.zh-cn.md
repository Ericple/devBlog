+++
date = '2025-02-13T15:01:30+08:00'
draft = false
title = '使用多线程'
tags = ['新手', '文档']
+++

Log4a允许开发者使用多线程来输出日志到文件，以获得最佳的性能

> [!WARNING]
> 由于设计缺陷，如果您使用的Log4a版本低于`1.3.0`，请避免使用多线程，这会导致性能降低。

## Worker带来的性能提升

在Log4a的Benchmark测试中，使用多线程的日志输出效率要比使用单线程高约2~7倍

![](/multiThread-logging-compare.svg)