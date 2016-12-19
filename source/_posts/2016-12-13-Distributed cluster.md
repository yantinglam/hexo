---
title: 分布式集群
date: 2016-12-13 14:00:00
categories:
- 技术
tags:
- 分布式
---
### 什么是分布式集群？
&emsp;&emsp;首先看看分布式与集群之间的区别与联系：
* 分布式是指将不同的业务分布在不同的地方；而集群是指将同一个业务部署到多台机器上
* 分布式的组织比较松散；而集群有一定的组织
* 分布式每一个节点完成不同的业务，一旦一个节点宕掉了，在这个节点上的业务就不可访问了；而集群里一旦某一台服务器宕掉了，其他的服务器可以顶上来
* 分布式的每一个节点都可以做成一个集群
<!-- more -->

&emsp;&emsp;所以分布式集群就是结合了分布式与集群的特征，相互取长补短，构建一个更加高效可用的系统模型。

### 为什么要使用分布式集群？
&emsp;&emsp;在实例生产中，单独使用分布式或单独使用集群的意义都不太大，分布式解决了应用服务业务或功能上的松耦合，但不能保证系统的高可靠性；而集群解决了应用服务处理高并发的能力，但不能很好的做应用服务的业务层扩展。所以分布式与集群应该是相辅相成、相互合作的关系。

### 如何使用分布式集群？
&emsp;&emsp;在设计分布式集群系统的时候要注意以下几点：
* 减少网络通信的开销
<p class="li-explanation">&emsp;&emsp;因为分布式集群的各个服务器之间的通信是通过网络进行，所以网络是分布式集群的一个必要条件。而目前网络的传输速度还赶不上CPU读取内存或硬盘的速度，这会成为系统性能提升的一个瓶颈点。为了减少这方面带来的影响，在设计上应该减少各个业务或功能模块之间不太必要的通信。</p>
        
* 将应用做成无状态的
<p class="li-explanation">&emsp;&emsp;首先搞清楚应用服务的状态指的是运行时程序因处理服务请求而存在内存里的数据。为什么要设计成无状态呢？是由于用户发起请求是对某一台服务器发起请求的，而这些用户请求信息只会存在于该台服务器上，如果此时该台服务器出现故障宕掉了，虽然其他服务器可以顶上代替该台服务器让应用继续运行，但是保存在故障服务器上的用户请求数据必然导致丢失，这就违背了高可靠的特性了。</p>

### 使用分布式集群应该注意的地方
* 网络性能的瓶颈
* 服务器数量越多，宕机的概率就越大
* 数据一致性的处理
<p class="li-explanation">&emsp;&emsp;因为在数据库的集群搭建是将多个数据备份部署到多个服务器上面，那么如何同步各个服务器之间的数据，让它们保持一致性就是一个必须要考虑的问题。</p>
          
### 分布式集群的设计原则
&emsp;&emsp;分布式集群的设计遵循CAP理论。所谓CAP理论是一个分布式系统最多只能同时满足数据一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）这三项中的两项。

#### 数据一致性的解决方案
* 强一致性：更新过的数据都能被后续的访问看到
* 弱一致性：能容忍后续的部分或全部访问看不到
* 最终一致性：经过一段时间后能访问到

#### 数据可用性
&emsp;&emsp;数据可用性是指对数据的读与写永远有效，这与数据冗余和负载均衡有着很大的联系。

#### 分区容错性
&emsp;&emsp;分区容错性是指分布式系统在遇到某节点或分区出现故障时，仍然能对外提供一致性和可用性的服务。