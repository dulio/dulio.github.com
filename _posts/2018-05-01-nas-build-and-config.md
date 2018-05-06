---
layout: post
title:  "自组NAS的折腾历程"
subtitle: ""
keyword: "nas,openmediavault,omv,omv4,docker,j4105,htpc"
date:   2018-05-01
categories: nas
tags:	nas htpc
background: ""
---

# 为什么写这篇博客

炫技！初衷也没有想做一个大而全的攻略，只想分享个人历程，给其他同学提供一个借鉴思路。

### 为什么要搭NAS

自搭NAS的好处懂的同学自然懂，简单来说就是：共享、便捷、安全。

- 备份：平时可以同步备份一些照片、小视频:>、大电影、下载等乱七八糟的东西，对程序猿来说，存一些零碎的配置文件，出门不会丢三落四，不要太爽

- 兼职HTPC：配合一些性能稍好的CPU直接硬解4k，取代专用的HTPC，节省成本

- 兼职服务器：搭一些奇奇怪怪的服务，常见的如gitlab、Nexus、Leanote等，还有小众的服务如Wekan，能用开源方式实现一大堆功能，数据存在自己的服务器上更放心

- 虚拟机服务器：NAS的CPU大多支持虚拟化，能跑一些玩的系统

---

# 硬件筹备

只打算讲重要的两个：CPU和机箱（毕竟也不拿广告费，哈哈）。

### CPU（主板）

一段时间史：

- 手头有一台老的NAS机器，CPU是D525（Atom），性能比较差，[CPU Benchmarks](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Atom+D525+%40+1.80GHz&id=611)，开机比较慢，跑起docker也比较费劲。当HTPC就不要想了。

- 当前比较流行的CPU主要是J3455，[CPU Benchmarks](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Celeron+J3455+%40+1.50GHz&id=2875)

- 2017年底牙膏厂发布了J4105（可以直接忽略J4205了）,[CPU Benchmark](https://www.cpubenchmark.net/cpu.php?cpu=Intel+Celeron+J4105+%40+1.50GHz&id=3159)

J4105优点：

- 原生4k@60hz：与上一代J4205一样，支持到4k@60hz，但是据说hdmi为原生，支持bt.2020，HDR10（没有仔细考究）。

- 支持DDR4：毕竟NAS是个相对低性能的平台，提升内存性能也会有比较高的性价比

总体来看，J4105在维持低功耗的同时，也能提供相对较好的性能，做HTPC靠谱一点。而且按照电子产品买新不买旧原则，我选择了J4105。

至于主板，截止目前（5月1日）还只听说过华擎出了相应产品（J4105-ITX，翻白眼），这款主板目前来看比较明显的缺点：

- 官方文档说仅支持最高8G内存（4Gx2），未测试

- 板载只有4个SATA口（通过PCI-E x1加扩展卡还能扩展出若干，但是受限于x1的带宽，肯定是存在瓶颈的）

- 主板仅支持UEFI，对linux系统安装会有一些阻碍（实测，可以关闭UEFI Secure，没有遇到啥阻碍）

### 机箱

选择要点是：

- 静音：不管做什么，静音总是最重要的

- 硬盘位多：别忘记本职

- 颜值高：毕竟要兼职HTPC

- 尽可能小：只能排最后了

真是组NAS的核心难题。。。

各种机箱之间也差异较大。主要分为以下几类：

- 品牌NAS机箱：专用的热插拔硬盘架，颜值也比较在线，但是性价比不一般的低。没错，说的就是你，万向。PASS

- 定制的工控机箱：价格优势明显，盘位一般也比较多，但是噪音、颜值、体积全下线。PASS

- HTPC机箱：寻找折中方案，在颜值比较高的HTPC里去寻找盘位尽可能多的产品。

最后，是在HTPC机箱里找到了一个款比较合适的。。。

### 组装

<img src="/assets/images/20180502234353.jpg" style="width:100%" alt="主板上电测试" title="主板上电测试">

主板上电测试（上图）

<img src="/assets/images/20180506134248.jpg" style="width:100%" alt="装机测试内部" title="装机测试内部">

装机测试内部（上图）

---

# 网络

网络这块单独拿出来，甚至放在操作系统的前面，是因为后面NAS很多功能都依赖于外部访问，如备份、服务提供。而且通常来说，一个家庭的网络环境通常无法变更，却对后续NAS功能有巨大的影响

下面主要针对如何具体情况给出方案。

## 家庭网络常见类型

- 有公网IP：如ADSL/FTTH(光纤到户)，特点是主路由器上的WAN IP是一个外网地址，可以在外网直接访问到

- 无公网IP：如FTTB(光纤到楼)，特点是主路由器上的WAN IP是一个内网地址，因为NAT，外网无法直接访问

### 有公网IP（Good luck，man）

这种情况一般出现在老小区，路由器PPPoE拨号时分配到一个随机的公网IP，这个IP能在任何地方访问。

我们只需要把NAS机器上的服务端口映射到路由器上，那么服务就连接到了公网。该方式提供的网络速度=网络上行速度（具体要见各地运营商的策略）。唯一需要解决的问题是如果寻找到这个动态的公网IP地址。

注：运营商基本都会禁止访问80/443端口

方案：

绑定域名并且使用dnspod API定期刷新域名解析。[参考](https://github.com/migege/dnspod)

### 无公网IP（I am sorry for that）

在新式小区一般才会采用这种方式（运营商成本较低）

这时，一般都需要寻找一台公网服务器，通过端口映射的方式，把本地服务接入外网，常见方案有：

- 花生壳（需要收费服务）

- 私有公网服务器+ssh/frp端口反向映射。

无论如何，都需要额外的成本，并且注意网络攻击与远程服务器的数据安全。

方案：

弄个国内云主机（挂域名更佳，不过要备案。。），家里使用梅林路由器，在路由器上搭建frp插件，充当内网与公网的桥梁，管理端口映射关系。

当然也可以在NAS机器上直接跑frp，但是在路由器上使用，在多NAS时（未来剁手预定）更容易管理维护

---

# 操作系统

目前能想到的主流选择是黑群晖、OMV(openmediavault)。

总结一下个人的需求，转化为技术指标的话有以下几点必需要考虑：

- 可靠的数据同步备份方案

- 管理面板远程访问、服务端口映射到外网的实现方案

- 远程下载，能支持普通、BT、电驴等下载

- docker、虚拟机必须支持

群晖是集成方案，除了虚拟机无法实现，其他基本都有了比较完善的方案。但是因为黑群晖没有官方服务与技术支持，遇到了一些问题（如突然无法打开docker终端）。所以放弃群晖。

OMV优点：

- 基于debian，对于玩惯了Linux系统的人来说，基本没有实现不了的东西。

- OMV的整体配置比较底层，能更好的控制系统/组件的运行

OMV缺点：

- 每个linux实现不了的功能都需要琢磨一套可行的开源方案

总之，OMV更符合我的预期。所以后面NAS会基于OMV。至于群晖，里面有太多兼容性问题，后面就不再讨论了。

## OMV4安装过程

OMV提供启动盘下载，[下载地址](https://www.openmediavault.org/download.html)
。

但是本人作为资深炫技与轻微技术洁癖人员，仗着熟悉Debian，打算使用debian 9 (Stretch)上升级omv的方式安装。

另外为了安装系统，准备了一个普通的4g U盘作启动盘和一个MLC芯片的32G U盘当作系统盘（正所谓铁打的数据，流水的系统）

### 启动盘准备

写入镜像：插上U盘，刻入Debian Stretch镜像

### Debian系统安装

插上启动U盘与系统U盘，引导启动U盘，开始安装！

<img src="/assets/images/20180506135911.jpg" style="width:100%" alt="开始安装" title="开始安装">

开始安装

<img src="/assets/images/20180506140111.jpg" style="width:100%" alt="注意安装分区" title="注意安装分区">

安装时盘比较多，需要注意安装到正确的系统U盘上

后面的安装过程不详述，仔细讲的话又可以写一篇文章 :)

### OMV安装

现在直接登录到安装好的Debian系统

先加源：

```
cat <<EOF >> /etc/apt/sources.list.d/openmediavault.list
deb http://packages.openmediavault.org/public arrakis main
EOF
```

安装：
```
apt-get update
apt-get --allow-unauthenticated install openmediavault-keyring
apt-get update
apt-get --yes --auto-remove --show-upgraded \
	--allow-downgrades --allow-change-held-packages \
	--no-install-recommends \
	--option Dpkg::Options::="--force-confdef" \
	--option DPkg::Options::="--force-confold" \
	install postfix openmediavault
```

初始化系统：
```
omv-initsystem
```

也可以看教程：[Install OMV4 on Debian 9](https://forum.openmediavault.org/index.php/Thread/21234-Install-OMV4-on-Debian-9-Stretch/)

安装完成后，就可以打开Web管理页面了

<img src="/assets/images/20180506142033.png" style="width:100%" alt="注意安装分区" title="注意安装分区">

登录页面，默认用户密码admin/openmediavault

### OMV基本配置

插件服务支持

```
wget http://omv-extras.org/openmediavault-omvextrasorg_latest_all4.deb
dpkg -i openmediavault-omvextrasorg_latest_all4.deb
```

### 磁盘分区与共享目录

<img src="/assets/images/20180506151130.png" style="width:100%" alt="物理磁盘" title="物理磁盘">

确认物理磁盘，并且使用Wipe格式化分区（注意备份数据！！！）

<img src="/assets/images/20180506151321.png" style="width:100%" alt="RAID分区" title="RAID分区">

创建Linux RAID分区，这里因为只有两块盘，采用Mirror镜像（RAID 1）方式

<img src="/assets/images/20180506151501.png" style="width:100%" alt="LVM分区" title="LVM分区">

创建LVM分区，最终创建出需要的逻辑分区。

<img src="/assets/images/20180506152510.png" style="width:100%" alt="挂载分区" title="挂载分区">

挂载分区

<img src="/assets/images/20180506152156.png" style="width:100%" alt="共享目录" title="共享目录">

分别在分区上建立共享目录

可参考的分区：

- docker-lib：高保护级别。存储docker镜像、容器信息

- docker-data：高保护级别。存储docker持久化信息

- virtualbox：高保护级别。因为想玩一下虚拟机，专门划出来存放虚拟机镜像与文件

- storage：不保护或者低保护级别。主要存放不重要的文档、媒体文件。

注：因为OMV在服务上挂载共享目录时，会改写目录权限，所以要分别建立共享目录，防止目录权限冲突。

---

# 常用服务搭建

### Docker

<img src="/assets/images/20180506150211.png" style="width:100%" alt="开启Docker源" title="开启Docker源">

开启Docker源

Plugin菜单开启DOCKER-UI插件

<img src="/assets/images/20180506150425.png" style="width:100%" alt="开启Docker源" title="开启Docker源">

开启Docker组件

<!-- <img src="/assets/images/20180506150837.png" style="width:100%" alt="开启Docker源" title="开启Docker源"> -->

开始使用Docker！

### VirtualBox

<img src="/assets/images/20180506153101.png" style="width:100%" alt="开启Docker源" title="开启Docker源">

很有意思，通过phpvirtualbox在Web管理，创建后可以通过Virtualbox的Remote Display打开远程桌面安装与使用。

主要用于安装Windows系统，干一些linux上干不了的事情。

实际使用体验（Win7，分配1核，2G内存）：略卡。。。好吧，看来J4105也只能勉强带起来

### FTP

<img src="/assets/images/20180506152820.png" style="width:100%" alt="开启Docker源" title="开启Docker源">

挂载共享目录，就可以让用户通过FTP访问

### 其他

其他如NFS、SMB、下载等都是开箱即用，没有什么好说。

并不是苹果粉，Time Machine什么的没研究方案。

下载功能只有基本的，youtube-dl组件也只能挂代理用，实在不行，只能开虚拟Windows解决（无奈脸）

---

# 总结

整体搭建下来的感觉是，无休止的折腾，必须要在安全与便捷、定制与易用之间做艰难选择。

目前整体搭建起来后，还比较满意，后面会继续进行NAS应用与HTPC方面的探索。

---

# FAQ

### NAS和普通PC比优势在哪里？

重要的事情说三遍：省电！省电！省电！

实现PC的功能，却只需要20W ~ 30W的电量，一年放着也用不了多少电费（精打细算脸）。

### 为什么不用白群晖？

太贵。。。

### 磁盘分区方案怎么选择？

目前流行的解决方案:

- ZFS：基于成熟的Oracle技术，拥有最多的特性，集成RAID技术，应该是首选的方案。虽然因为协议问题没有在Debian的主库中，但是仍然可以使用。

- LVM2 + Linux RAID：基于Linux维护的软件与库，使用最多的方式。

- MergerFS + SnapRAID：构建于底层分区体系上的方案，没有实际使用。

最初通过在OMV上安装ZFS插件，尝试使用ZFS，但是OMV对ZFS支持不佳，应该是设计缺陷，无法在ZFS分区上创建共享目录（Shared Folders），导致无法在GUI上方便使用，于是无奈弃用。。。

关于MergerFS + SnapRAID方案，因为是较新的方案，还未调研功能性与稳定性，不做评论

最后的选择是使用最常见的LVM2 + Linux软RAID。
