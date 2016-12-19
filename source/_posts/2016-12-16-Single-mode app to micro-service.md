---
title: 从单体式架构迁移到微服务架构
date: 2016-12-16 16:02:00
categories:
- 技术
tags:
- 微服务架构
---
&emsp;&emsp;从单体式架构迁移到微服务架构不应该通过从头重写代码的方式实现，而是应该通过逐步迁移的方式。
<!-- more -->
### 停止挖掘
&emsp;&emsp;停止挖掘就是将新功能以一个独立的微服务方式实现，而不是往旧单体应用上继续添加代码，新的微服务通过接口接入到旧的单体应用中。

&emsp;&emsp;除了将新功能开发为新服务之外，还需要增加两个模块来将旧单体应用与新服务连接起来，这两个模块分别是：
* 请求路由器
<p class="li-explanation">&emsp;&emsp;负责处理入口（http）请求，将新功能的请求发送给新服务，将传统请求发送到传统单体应用。</p>

* glue code（胶水代码，也称为容灾层）
<p class="li-explanation">&emsp;&emsp;将微服务和单体应用集成起来，保护微服务全新域模型免受传统单体应用域模型的污染；而且还负责数据整合，提供两种模型间的翻译功能<br/>&emsp;&emsp;glue code可以是在单体应用处，可以是在微服务处，也可以两都兼而有之。</p>

<img src="/images/Single-mode app to micro-service/Add new service to single-mode app.png" width=400 height=400 />

#### 停止挖掘的优点：
* 阻止单体应用变得更加的无法管理
* 微服务本身可以开发、部署和独立扩展

### 前后端分离
&emsp;&emsp;前后端分离是将表现层与业务数据访问层分离，通过若干方面组成的粗粒度APIs将单体业务分割成两个更小的应用。
<img src="/images/Single-mode app to micro-service/Separating front-end and back-end.png" width=600 height=600 />
#### 前后端分离的好处：
* 使得应用分成两部分进行开发、部署和扩展
* 允许表现层开发者在用户界面上快速选择，进行A/B测试
* 一些远程API可以被微服务调用

### 抽出服务
&emsp;&emsp;将现成的模块抽取变成微服务。

#### 模块抽取的顺序
* 根据获准程序排序，先抽取获益最大的，一般从经常变化的模块开始获益最大
* 根据资源消耗程度排序，一般先抽取资源消耗最大的开始
* 根据现有粗粒度边界来排序，将拥有粗粒度边界的两部分拆分

#### 抽取步骤
1. 定义好模块和单体应用之间粗粒度接口
2. 将模块转换成独立服务
3. 开发、部署和扩展另外一个服务

<br/>
参考链接：
[微服务实践（七）：从单体式架构迁移到微服务架构](http://dockone.io/article/1266)
