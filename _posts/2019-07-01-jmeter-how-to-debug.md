---
layout: post
title:  "JMeter的源码调试"
subtitle: ""
keyword: "jmeter,source,debug"
date: 2019-07-01
categories: jmeter
tags: jmeter,source,debug
background: ""
---

# 1. 背景

JMeter是一套开源的性能测试工具。因为基于Java语言开发、支持丰富协议、原生支持分布式等特性使JMeter成为非常流行的性能测试工具。

得益于简单的线程策略与长期的开发，JMeter已经相当稳定成熟。所以在性能测试平台上使用了JMeter作为发压引擎。

在对JMeter的使用中，会逐渐产生一些扩展JMeter的需求，从而需要对JMeter的内部机制有一定的了解。然而JMeter使用了Ant这种略古老的版本管理器，所以有了这篇文章，记录调试时的一些经验

# 2. 调试配置

## 2.1 准备环境

下载JMeter源码，地址 `https://github.com/apache/jmeter.git`

下载安装ant，或者使用IDEA内置ant

## 2.2 代码导入

IDEA打开JMeter源码

使用JDK8，language level设置为8

因为ant项目，导入IDEA时无法自动识别代码路径，需要手动配置：

source目录：

- src/components 

- src/core 

- src/examples 

- src/functions 

- src/jorphan 

- src/junit 

- src/protocol 

test目录：

- test/src

excluded目录（排除目录）：

- build

## 2.3 依赖管理

打开IDEA的Ant，执行download_jars (即`ant download_jars`)

在项目配置中的Libraries添加以下路径：

- lib
- lib/doc
- lib/ext
- lib/junit
- lib/opt

## 2.4 debug配置

在Run/Debug配置中添加一条Remote Debug，端口假设为5005，得到以下配置`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005`

找到build.xml，找到run_gui这个任务，把上述调试指令交入jvm参数，如:
```xml
<jvmarg value="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"/>
```

## 2.5 启动调试

启动ant run_gui任务，同时打开remote debug，开始调试

# 总结

JMeter在IDEA上的调试方式没有在文档上说明，对Ant这种古老的管理工具也不太熟悉，总体来说是摸索出来的。

看来下次得再看看时新的压测工具，如gatling这类较新的工具。后面再对JMeter的一些源码进行分析。