---
title: 微服务架构的IPC
date: 2016-12-15 15:07:08
categories:
- 技术
tags:
- 微服务
---
&emsp;&emsp;IPC即进程间通信（Inter-Porcess Communication）。微服务架构中有两类IPC机制可选：异步消息机制和同步请求/响应机制。<!-- more -->

### 异步消息机制：
&emsp;&emsp;异步消息机制是基于消息进行通信的。一个消息由头部（元数据，如发送方信息）和消息体构成，消息通过channel发送。

&emsp;&emsp;任何数量的生产者都可以发送消息到channel中，同样任何数量的消费者都可以从channel中接受数据。

#### Channel的种类：
* 点对点channel
<p class="li-explanation">&emsp;&emsp;把消息准确地发送到某个会从channel中读取该消息的消费者。</p>
* 发布/订阅channel
<p class="li-explanation">&emsp;&emsp;把消息投送到所有从channel中读取该消息的消费者。</p>

#### 异步消息机制的优点：
* 解耦客户端与服务端
<p class="li-explanation">&emsp;&emsp;客户端只需要将消息发送到正确的channel上，完全不需要了解具体的服务实例。</p>
* 消息缓冲
<p class="li-explanation">&emsp;&emsp;消息协商器将所有写入channel的消息按照队列方式管理，直到被消息者处理，这样的话，只要保证消息进入到消息队列中，即使系统很慢甚至暂时不可用也没关系，因为队列中会一直保存着这个消息。</p>
* 弹性的客户端-服务端交互
* 直接的进行进程通信

#### 异步消息机制的缺点：
* 额外的操作复杂性
<p class="li-explanation">&emsp;&emsp;消息系统需要单独安装、配置和部署。消息协商器必须要是高可用的。</p>
* 实现基于请求/响应交互模式时会变得比较复杂
<p class="li-explanation">&emsp;&emsp;在异步消息机制中，每个请求消息必须包含一个回复渠道ID和相关的ID，处理基于请求/响应交互的模式时，服务端发送一个包含相关ID的消息到channel中，使用相关ID来响应对应发出请求的客户端。如此一来，使用一个直接支持请求/响应的IPC机制会更容易些。</p>

### 同步请求/响应机制：
&emsp;&emsp;同步请求/响应是客户端向服务端发送一个请求，服务端处理请求，然后返回响应。

#### RESTful：
&emsp;&emsp;首先了解一下什么是REST？REST是一个资源，一般代表一个业务对象。

&emsp;&emsp;RESTful API是基于HTTP的一种协议，使用HTTP语法协议来修改资源。

##### RESTful API成熟度模型：
* Level0的web服务只是使用HTTP作为传输方式，实际上只是<a name="RPC" href="#RPC-explanation">RPC</a>用的一种具体形式
* Level1的web服务引入了资源的概念，每个资源有对应的标识符和表达
* Level2的web服务使用不同的HTTP方法进行不同的操作，并使用HTTP状态码来表示不同的结果
* Level3的web服务使用<a name="HATEOAS" href="#HATEOAS-explanation">HATEOAS</a>，在资源的表达中包含了资源的链接信息，客户端可以根据链接来发现可执行的动作

##### 使用HTTP的优点：
* 简单且大家都熟悉
* 可使用浏览器扩展（如postman）或curl之类的命令来测试
* 内置支持请求/响应模式的通信
* 对防火墙友好
* 不需要中间代理，简化系统架构

##### 使用HTTP的缺点：
* 只支持请求/响应模式交互
* 因为客户端与服务端直接通信，交互期间必须一直在线
* 客户端必须知道每个服务实例的URL


#### Thrift：
&emsp;&emsp;Thrift是Facebook实现的一种高效的、支持多种编程语言的远程服务调用框架。

&emsp;&emsp;Thrift可以返回响应，返回值的方法其实就是请求/响应类型交互模式的实现。

&emsp;&emsp;Thrift可以对应于通知类型的交互模式，定义为单向的，此时服务端将不返回响应。

### 消息格式
&emsp;&emsp;在进程间通信必须选择一个能跨语言的消息格式。有两类格式：
* 文本：如JSON、XML
<p class="li-explanation">&emsp;&emsp;这类格式的优点在于可读性强，而且是自描述的。缺点就是消息会变得冗长，以及解析文本的负担过大。</p>
* 二进制
<p class="li-explanation">&emsp;&emsp;这类格式的优点在于解码速度更快，效率更高。缺点在于可读性差。</p>

<br/>
注释：
<a name="RPC-explanation" href="#RPC">RPC：</a>Remote Procedure Call Protocol - 远程过程调用协议
<a name="HATEOAS-explanation" href="#HATEOAS">HATEOAS：</a>HyperMedia As The Engine Of Application State - 超媒体作为应用状态的引擎

<br/>
参考链接：
[微服务实战（三）：深入微服务架构的进程间通信](http://dockone.io/article/549)