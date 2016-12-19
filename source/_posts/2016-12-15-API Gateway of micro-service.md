---
title: 微服务架构中的API Gateway
date: 2016-12-15 14:07:08
categories:
- 技术
tags:
- 微服务架构
---
### 什么是API Gateway?
&emsp;&emsp;API Gateway是客户端与微服务系统之间的中间件，是客户端访问微服务系统的唯一入口。<!-- more -->打个比方，将微服务系统比作一个守卫深严的机密图书馆，将客户端比作图书馆的来访人员，API Gateway就好比是图书馆管理员，这个图书馆里的图书只能由管理员来查阅，所以来访人员想要查询任何图书馆内的资料，都要经过管理员。

&emsp;&emsp;来访人员（__客户端__）要想查询（__访问__）图书馆（__微服务系统__）里的某些资料（__服务__），就要先告诉管理员（__API Gateway__）他想要查询的信息是什么，然后由管理员去翻阅图书馆里的资料，有的可能翻阅一本资料就得到答案，有的可能需要翻阅好几本资料才能得到答案，但这个来访人员不需要知道。最后管理员整理好最终答案后告诉来访人员。

&emsp;&emsp;在这个过程中，来访人员只需要关注两点：他想要查询的信息是什么以及他得到的最终答案是什么。来访人员不需要知道得到这个信息的查阅过程，这部分是由管理员来负责的。

&emsp;&emsp;管理员需要懂得多方语言，才能读懂图书馆里各方语言的资料，也就是说API Gateway必须要有转换不同通讯协议的机制，以便与微服务系统中各个使用不同语言工具的服务进行通讯连接。

<img src="/images/API Gateway of micro-service/Model of API-Gateway.png" width=500 height=500 />

### API Gateway的功能
* 授权机制
<p class="li-explanation">&emsp;&emsp;管理员可以决定来访人员是否有资格进行信息查询。</p>
* 监控机制
<p class="li-explanation">&emsp;&emsp;管理员可以知道图书馆里资料的使用情况。</p>
* 负载均衡
<p class="li-explanation">&emsp;&emsp;管理员可以对图书馆里的资料进行分配使用。</p>
* 缓存
<p class="li-explanation">&emsp;&emsp;管理员可以记住那些常被访问的资料内容，以便能快速回答来访人员的问题。</p>
* 请求分片和管理
<p class="li-explanation">&emsp;&emsp;管理员可以将来访人员要查询的信息分好几次来进行查询。</p>

### API Gateway的优点
* 封装微服务系统的内部结构
* 减少客户端与微服务系统的通讯次数
* 使客户端不需要处理不同服务之间不同的通讯协议，简化客户端代码

### API Gateway的缺点
* API Gateway本身必须要是一个高可用的组件，所以对它的开发、部署和管理要求较高
* 每次新增服务时，也需要更新API Gateway来支持新的服务
* API Gateway的更新方法越简单越好，否则开发者将因为更新API Gateway而要排队

<br/>
参考链接：
[微服务实战（二）：使用API Gateway](http://dockone.io/article/482)