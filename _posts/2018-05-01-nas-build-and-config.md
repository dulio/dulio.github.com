---
layout: post
title:  "NAS自搭建与配置"
subtitle: ""
keyword: "nas,j4105,htpc,"
date:   2018-05-01
categories: nas
tags:	jekyll github blog
background: '/img/posts/06.jpg'
---

# 为什么要干这个事情

自搭NAS的好处懂的同学自然懂，简单来说就是：共享、便捷、安全。

- 备份：平时可以同步备份一些照片、小视频:>、大电影、下载等乱七八糟的东西，对程序猿来说，存一些零碎的配置文件，出门不会丢三落四，不要太爽

- 兼职HTPC：配合一些性能稍好的CPU直接硬解4k，取代专用的HTPC，节省成本

- 兼职服务器：搭一些奇奇怪怪的服务，常见的如gitlab、Nexus、Leanote等，还有小众的服务如Wekan，能用开源方式实现一大堆功能，数据存在自己的服务器上更放心

- 虚拟机服务器：NAS的CPU大多支持虚拟化，能跑一些玩的系统

### 优势在哪里？

重要的事情说三遍：省电！省电！省电！

实现上述的功能，却只需要20W ~ 30W的电量，一年放着也用不了多少电费（精打细算脸）。

### 为什么不用白群晖？

太贵。。。

# 硬件筹备

只打算讲重要的两个：CPU和机箱（毕竟也不拿广告费，哈哈）。

### CPU（主板）

一段时间史：

- 手头有一台老的NAS机器，CPU是D525（Atom），性能比较差，[CPU Benchmarks](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Atom+D525+%40+1.80GHz&id=611)，开机比较慢，跑起docker也比较费劲。

- 当前比较流行的CPU主要是J3455，[CPU Benchmarks](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Celeron+J3455+%40+1.50GHz&id=2875)

- 2017年底牙膏厂发布了J4105（可以直接忽略J4205了）,[CPU Benchmark](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Celeron+J4105+%40+1.50GHz&id=3159)

J4105优点：

- 原生4k@60hz：与上一代J4205一样，支持到4k@60hz，但是据说hdmi为原生，支持bt.2020，HDR10（没有仔细考究）。

- 支持DDR4：毕竟NAS是个相对低性能的平台，提升内存性能也会有比较高的性价比

按照电子产品买新不买旧原则，我选择了J4105。

至于主板，截止目前（5月1日）还只听说过华擎出了相应产品（J4105-ITX，翻白眼）

# 软件
