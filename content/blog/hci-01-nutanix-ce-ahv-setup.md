+++
date = 2022-01-03T18:19:20+08:00
title = "HCI | 超融合平台线上训练营第一期"
description = "超融合技术线上学习训练营开营，用超融合解决当前的环境资源短缺问题。"
author = "Martin Liu"
categories = ["DevOps"]
tags = ["hci", "nutanix", "超融合"]
[[images]]
  src = "https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-th1.jpg"
  alt = "流程设计"
  stretch = "vertical"

+++

本期分享的内容是超融合的基础知识，Nutanix CE社区版AHV/KVM单节点集群的搭建。

<!--more-->

在本期线上直播中，我们给大家科普了超融合的基础概念，介绍了Nutanix CE社区版【AHV/Kvm虚拟化】的单机安装和基础配置。本文为大家回顾一下其中的技术要点。

观看本次直播并且中奖的朋友们是：我是ZCQQQ、盔甲小子、繁硕星辰、壮星豆、flyinghourse110、YS8297和阿尔萨狮123，请联系微信 Jane-happylife 索取你的奖品。本次奖品是Nutanix提供的空气加湿器或者运动背包。备注 12月超融合直播活动 领取礼品。两种礼品（根据库存来决定随机寄出）。

本期的视频回放链接在这里：https://www.bilibili.com/video/BV1YY411a7w2/

## 认识 HCI 超融合

来自网络的定义：超融合基础架构（Hyper Converged Infrastructure，或简称“HCI”）是指在同一套单元设备中不仅仅具备计算、 网络 、存储和服务器虚拟化等资源和技术，而且还包括备份软件、 快照技术 、重复数据删除、在线数据压缩等元素，而多套单元设备可以通过网络聚合起来，实现模块化的无缝横向扩展（scale-out），形成统一的资源池。

从我个人角度看，我的对基于OpenStack的私有云持有消极的态度。虽然，OpenStack 给企业数据中心基础设施的建设路线带来了不小的影响；但是，它还是无法解决传统数据中心里经典的三层架构基础设施的三大根本痛点。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%872.JPG)

三层架构即由x86服务器层、网络层（以太网/SAN）和存储层所搭建的规模不大数量巨多的虚拟化/物理机集群资源池。由于虚拟化技术的成熟和普及，这些虚拟化集群资源池为业务应用提供了所需要的计算、存储和网络资源。在最近十几年的企业数据中心中处于主力架构，但是它有其核心的三大问题：

1. 性能瓶颈问题比较突出：当突发大量来自于网路的业务请求时，且处理它们需要大量的读写存储操作时，后台存储系统乃至SAN网络都将发生性能耗尽的瓶颈，存储延时剧增，等待时长暴增，还可能伴随大量读写操作失败。
2. 容量无法线性扩展：三层资源中的任何一层的资源容量耗尽后，都需要多个专业运维团队进行大量手工操作，才能进行物理量容量的扩容，扩容过程中很难实现业务服务的持续性，数据量大的时候甚至不得不面临很长的业务停机时间窗口。
3. 复杂性令运维和开发胆寒：虽然在虚拟化系统中，虚拟化技术可以提供资源层的HA/FT等功能，但是当业务出现宕机故障时，从应用到下层的硬件，从上到下排查的时候，所有相关团队都不得不全员上线救火，业务服务故障很难在这个复杂性极其高的系统中快速定位。服务质量水平命悬一线，而且SLA很难挽救。

超融合的理想是全面消除以上三个问题。超融合系统其实为三层架构做了减法，它对传统磁盘阵列的存储设备产生了颠覆性的冲击，它利用优化的网络分布式存储服务消除了传统架构中的存储层。它还利用自动化部署工具，极大的降低了虚拟化集群的搭建、升级和维护过程。HCI超融合系统的本身及其精简，非常好扩容和维护，而且可以实现容量和性能的同步线性按需增长。从云计算的五个基础特征来考察HCI技术栈，我个人觉得它比OpenStack更像是云计算。

学习超融合技术栈也有很多种学习路径，动手搭建超融合系统无疑时最佳的方式。本期超融合视频中，我们选择的是免费版的Nutanix CE社区版。下面划出本期直播中的重要知识点。希望大家可以在自己的环境中开始超融合刷机之旅，当超融合成为你的一个日常工具的以后，你可能才会对HCI有更深的理解。

## 硬件选择

HCI系统是虚拟化、x86计算机、万兆以太网和超融合软件完美的组合。在你的工作环境中就存在着大量可用的机器，经过一定的挑选和准备之后，你就应该可以操练超融合技术了。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%873.JPG)

任何可以运行服务器虚拟化【esxi/kvm】软件的x86计算机都可以，最近10年以内采购的机器应该都可以适用。为了让其发挥应有的价值和意义，下面是一些建议选项：

* 最低配置：cpu 桌面机 i5/i7级别起步，4个物理核，或更高配置；服务器推荐E3/E5的多核处理器起步，或者更优CPU；内存32 GB，或更多。
* 建议配置：cpu尽量选择双路E5级别，或更新的CPU；单机内存建议128GB，或更多，或者按需增加。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%874.JPG)

磁盘选择是超融合系统的重点之一，通常包括SSD和机械磁盘，它是组成分布式存储服务的基础。对于使用RAID卡的服务器，RAID卡上需要删除所有盘的任何RAID配置，确保所有磁盘都直通给主板，SATA接口的主板，请直接按照先后顺序，先连接SSD磁盘，后连结机械盘。

* 最低配置：单机配置1块64GB SSD用于运行虚拟化软件（U盘-8/16GB也可以，但不建议），250GB SSD 一块或两块，用于安装和运行超融合软件，这是HCI的核心，它往往是以一个虚拟机的形式存在的，该虚拟机会完全运行在SSD上，SSD上的剩余空间计算在分布式存储池的全局空间中，SSD对虚拟机起到了重要的加速功能。机械磁盘一块或者多块，用于提供廉价大量的存储空间，机械磁盘的存储容量和SSD一起计入存储池的空间中。对于纯软件测试目的机械硬盘可以不要。SSD是必须的。机械盘是可选的。推荐混合使用，从而实现性能和空间的鱼和熊掌兼得。
* 建议配置：单机配置1块64GB SSD用于运行虚拟化软件，两块1TB的SSD用于运行超融合软件，且为全局存储池提供足够的高速性能SSD存储空间。两块2TB机械硬盘起步，机械盘后续还可以按需添加。集群中所有计算机上的所有SSD和机械磁盘共同组成了统一的混合型存储层。每个虚拟机都可以访问到所有磁盘中的性能和空间。Nutanix CE版文档声称支持总共四块磁盘，但是社区中也有朋友分享6+块盘的正常工作案例。

网络的选择有以下注意事项：

* 最低配置：单节点配置至少一个千兆网口，对于单机的超融合系统而言，Nutanix也可以利用到基本上大部分优势功能，包括：运行虚拟机、为虚拟机提供混合盘组成的存储空间、单节点上双副本的数据可靠性等等，我曾长期运行在这个配置上。
* 建议配置：一块双口万兆网卡，一个万兆网口用于集群内的存储网络流量，另外一个用于虚拟机中应用的相互网络访问，用VLAN隔离两个网段的流量。测试的目的下，可以混合在一个具备Internet访问条件的网段中。
* 建议优化配置：两块双口万兆网卡，一块双口千兆网卡，用于对网络流量、高可用性和性能的进一步优化，这里先不做展开。

网络交换机的选择，Nutanix CE免费版支持1、3、4节点的搭建模式。单节点没有特殊要求，多节点情况下，现场需要具备至少一台万兆以太网交换机和一台千兆以太网交换机。并且做相应的VLAN准备。

## 准备工作

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%875.JPG)

必须在 https://next.nutanix.com 注册社区账号，记录登录邮箱和密码备用。访问 [Download Community Edition | Nutanix Community](https://next.nutanix.com/discussion-forum-14/download-community-edition-38417) 这个帖子，下载 Installer ISO  https://download.nutanix.com/ce/2020.09.16/ce-2020.09.16.iso 。将这个ISO文件刻录在启动U盘中。安装前确保网线已经连结正常，且能访问Internet。

Nutanix CE安装文件也可以在这个网盘中下载：链接：https://pan.baidu.com/s/1DdvvfcPhcpR1jkfrXhCraA    提取码：4dgc 

## 安装过程

Nutanix CE版的安装可能会持续1小时左右，过程中包含比较多的自动化操作流程，基本需要用户少量的输入操作，其他更多的是等待时间。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%876.JPG)

下面是必须要关注的注意事项，相当于checklist：

1. 在主板BIOS的启动模式选项中，设置为legacy模式，其他模式不支持。
2. 在BIOS中确认能看到所有磁盘。
3. 插入一个桌面版 Linux Live CD U盘（如fedora 35），或者Win PE启动U盘，先设置从这个U盘启动。
4. 用Linux Live CD中的图形化磁盘分区工具，检查所有磁盘的空间，SMART信息，如果磁盘已经可见的老化或者错误，就替换掉在进行后续的装机。删除掉所有磁盘中的分区。重启。
5. 插入Nutanix CE社区版安装U盘。开始正式的刷机流程。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-disk.png)

选项介绍：

* 选中 AHV - 即Nutnaix版本的KVM的虚拟化软件。
* 按顺序选中：服务器虚拟化 Hpyervisor 的SSD盘，CVM的SSD盘，和做数据空间用的机械磁盘。
* Host ip ： 物理机运行虚拟化所在网段IP
* CVM ip：超融合软件运行的网段IP，必须通Internet。
* 不要选则创建单节点集群选项。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-license.png)

阅读完用户许可证全文后，选中接受，并且点击开始。Nutanix CE是一个裁剪版本有限制的软件，本文建议用于开发测试环境。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-hypervisor.png)

开始了物理机的虚拟化的安装，然后自动开始CVM的安装，CVM是一个虚拟机软件，会被自动启动和部分初始化。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-end.png)

最后，安装成功的话，拔出U盘，重启服务器。如果是多节点安装，每个节点都需重复这个过程，注意提前规划好Host Ip和CVM Ip的网段。

## 初始化配置

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%8711.JPG)

在安装完第一次重启的过程中，我们需要一些等待时间，逐步确认超融合系统的充分就绪，一下每个步骤必须顺序进行，不可跳跃进行：

1. ssh 登录 host 的ip 地址，root 的默认密码是 nutanix/4u ；登录后，使用hostname 命令查看并记录主机名，运行 virsh list 命令观察 cvm虚拟机的运行状态。
2. ssh 登录 cvm 的 ip 地址，默认用户名 nutanix ， 密码 是 nutanix/4u ；登陆后，使用 hostname 命令确认是否变为了 host的主机名+ -cvm 这样的名字，如果看到的是一个 cvm 的ip 地址，什么也不要做。等cvm第一次重启后，在此登录确认主机名。
3. 在cvm 中执行 ping qq.com 命令，确保和internet的互联。
4. 在cvm 中执行 genesis status 命令，如果过输出是持续报错信息，就等待错信息的停止，最后正常的输出应该是若干行服务的名称和端口号。
5. 执行 cluster status 命令，如果过输出是持续报错信息，就等待错信息的停止，知道最后一行显示本系统中还没有被配置，需要创建集群。
6. 执行超融合集群的创建命令， cluster -s 192.168.1.20 create ； 这里是创建单节点集群，ip地址就是当前cvm的ip地址。等待这条命令初始化集群完毕后，可以看到Nutanix超融合系统的所有服务状态清单。

多节点（3/4）节点 AHV 集群的搭建说明:

1. 重复以上的安装过程三次或者四次
2. 确保每个计算机上的cvm都处于 cluster status 输出信息正确一致的状态
3. 在每个 cvm上ping 其它邻居cvm的ip，确保网络畅通。
4. 在其中一个cvm的命令行执行多节点集群创建命令：  cluster -s 192.168.1.20,192.168.1.22,192.168.1.24 create 。该命令的正确结果是，每个cvm上的集群服务状态信息。

最后，在cvm上使用 cluster status 命令，确认所有节点的所有服务都正常运行了以后，继续登录超融合系统的网页图像界面。

## 集群使用基础

顺序只需一下基础的配置和功能操作。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%8713.JPG)

1. 在浏览器中访问任一cvm的ip地址，使用默认用户名 admin 密码 nutanix/4u登录，立刻会让你修改 admin 的默认密码，使用并记录这个新密码备用。
2. 用新密码登录。
3. 在要求输入Next 账号的地方，输入前面注册好的 Nutanix 论坛的邮箱和密码。
4. 成功登录后，你可以看到超融合的初始化界面，观察第一个关键信息点，观察左上方的第二个信息块，这里的显示的是存储池的物理和逻辑容量。如果这个数据没有立刻显示出来，那么表明该节点存在的某种问题，但是这不表明它不能用于多节点集群的创建。我有三个计算机时这种情况，单节点初始化失败的计算机在多节点集群的创建中反而没有问题。
5. 在右上角的配置菜单中，选中镜像访问 image service ，在哪里上传一个准备好的 iso，img 或者 qcow2 文件。等待几分钟后，每个文件应该即处于就绪使用的active 状态。
6. 在网络配置中创建一个用于虚拟机运行的网络。
7. 创建空白的虚拟机，挂在iso文件，并且开始虚拟机的安装。
8. 创建空白虚拟机，创建基于img 或者 qcow2虚拟机模板文件的磁盘，开机后即可使用。

## 参考成本

我个人喜欢在本地运行很多虚拟机的测试环境，属于 self-host 粉。我的虚拟机数量逐渐增加。从一个Intel NUC的单机微型测试机。 到单机联想工作站（双路E5CPU），到现在新增了两个某宝的组装兼容机，最后组成了三节点的Nutanix 超融合集群【vSphere 虚拟化】。个人实现多节点超融合集群的必要性因人而异，这里只是告诉大家，使用Nutanix CE在这些机器上是可行的。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-%E5%B9%BB%E7%81%AF%E7%89%871.JPG)

下面是网购订单的统计信息，希望对于想自行攒机的朋友，或者对扩容利旧服务器的朋友有些帮助。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-2021-12-29_12-32-49.png)

## 下期预告

HCI系统的搭建还有很多问题值得一起探讨和学习，HCI超融合系统软件还有很多功能可以学习了解。我们为DevOps社区的朋友们新开辟了这个HCI板块，希望以后会每个月为大家输出一期新的内容。下期直播的简介如下：

1. 日期：2022年1月26日中午1点
2. 内容1：Nutanix CE版AHV安装答疑解惑
3. 内容2：VMware vSphere 集群的搭建 
4. 主持人：刘征，吴孔辉
5. 直播平台：B站搜/关注：中国DevOps社区
6. 直播抽奖：未知
7. 报名：互动吧搜/关注：中国DevOps社区

## 其他

HCI超融合技术对于了解它的人来说很火热，但是还有很多不太了解HCI的人；HCI的用户对它也有不同的使用场景和期望。请大家参与这个《中国超融合状态调查》，我们每月为参与本调查的朋友进行抽奖，抽奖结果在下次直播的时候公布。扫码参与调查。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-dc.png)

我们为希望学习HCI技术的朋友们创建了一个qq群，希望可以给大家带来一定帮助。扫码加入我们的QQ群。

![](https://elasticstack-1300734579.cos.ap-nanjing.myqcloud.com/2022-01-04-qq.png)