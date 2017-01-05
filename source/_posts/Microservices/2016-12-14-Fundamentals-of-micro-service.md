---
title: 微服务架构的认识
date: 2016-12-14 14:07:08
categories:
- 技术
tags:
- 微服务
---
### 什么是微服务？
&emsp;&emsp;微服务是一种架构风格，在这种架构中，由一个或多个微服务组成一个大型应用或软件。一个微服务代表着一个小的业务功能，各个微服务之间是松耦合的，通过<a href="#IPC-explanation" name="IPC">IPC</a>相互通信。单个微服务可以被独立部署。
<!-- more -->
### 微服务架构与单体式架构应用对比
<table><tr><th></th><th>单体式</th><th>微服务</th><th></th></tr><tr><td>进程安排</td><td>将所有功能都放在一个进程中</td><td>将所有功能拆分为多个业务功能安排到多个服务进程中</td><td rowspan = 2><img src="/images/Microservices/Fundamentals-of-micro-service/Micro-service-and-single-mode-app1.png"></td></tr><tr><td>扩展方式</td><td>通过复制整个应用到多台服务器实现扩展</td><td>通过将不同的服务分布于不同的服务器，并按需复制服务实现扩展</td></tr><tr><td>数据管理</td><td>业务逻辑与数据进行分离，数据一般采用统一设计、统一管理的方式</td><td>将相关联的业务逻辑及数据放在一起形成独立的边界，每个服务管理自有的数据库，每个数据库可以根据该服务的需求有自己的设计</td><td><img src="/images/Microservices/Fundamentals-of-micro-service/Micro-service-and-single-mode-app2.png"></td></tr><tr><td>团队结构</td><td>按功能进行组织，如UI界面设计的一组，数据库的一组，中间件的一组...</td><td>跨功能进行组织，既有UI界面的，也有数据库的，还有中间件的...通常团队不会太大</td><td rowspan = 2><img src="/images/Microservices/Fundamentals-of-micro-service/Micro-service-and-single-mode-app3.png"></td></tr><tr><td>开发及运维方式</td><td>以项目模式开发完整的应用，开发完成后交付给运维团队负责维护</td><td>倡导一个团队负责一个“微服务”的完整生命周期，倡导“谁开发、谁运维”的开发运维一体化</td></tr><tr><td>技术平台</td><td>采用单一的技术平台，如一个单体式应用一般只采用一种编程语言</td><td>鼓励使用合适的工具完成各自的任务，可以选用各自最佳的工具，如不同的编程语言</td><td></td></tr></table>

<br/>
### 微服务架构与<a href="#SOA-explanation" name="SOA">SOA</a>架构对比
<table><tr><th></th><th>微服务</th><th>SOA</th></tr><tr><td>相同点</td><td colspan = 2><ul><li>以服务为核心</li><li>强调敏捷及快速推出市场</li></ul></td></tr><tr><td>差异点</td><td><ul><li>强调如何将一个整体的应用拆分成多个可独立部署的服务进程，并通过开放编程语言及数据持久化等措施最大化交付的速度</li><li>强调的“智能端点与管道扁平化”特性与<a href="#ESB-explanation" name="ESB">SOA</a>背道而驰</li><li>利用许多大型组织中应用的新技术与经验，尽可能使用简单的通讯协议如RESTful API</li></ul></td><td><ul><li>SOA内涵广泛，但主要场景包括应用系统的资产重用服务化、通过<a href="#BPM-explanation" name="BPM">BPM</a>实现服务编排，通过ESB实现不同应用的集成</li><li>SOA以Web Service/BPM/ESB等技术为主相对而言显得复杂些</li><li>SOA试图通过ESB隐藏集成的复杂性，却花费巨资而没有体现出价值性</li><li>过度集中管控而抑制了个性化变更</li></ul></td></tr></table>

<br/>
### 为什么要使用微服务架构
#### 微服务的优点
* 微服务架构是松耦合的，每个服务相对比较简单，只关注一个业功能务，可以提供更高的灵活性
* 微服务架构中的每个服务可由不同的团队独立开发，可根据自身业务选用适用的编程语言与工具进行开发，更能够有针对性地解决问题
* 每个服务可独立部署，这样可以加快部署，加快推出市场的速度
* 使用微服务，可以频繁地发布不同服务的同时保持系统其他服务的可用性和稳定性

#### 微服务的缺点
* 进程间通讯机制带来的挑战
<p class="li-explanation">&emsp;&emsp;需要在RPC或者消息传递之间选择并完成进程间通讯，需要处理消息传递中速度过慢或者不可用等局部失效问题。</p>
* 分区数据库带来的挑战
<p class="li-explanation">&emsp;&emsp;需要更新不同服务所使用的不同数据库，需要处理不同服务之间的数据访问问题。</p>
* 测试难度提高
<p class="li-explanation">&emsp;&emsp;测试一个服务需要启动和它有关的所有服务，而且在动态环境下，服务间的交互会产生非常微妙的行为，对于这种行为难以可视化及全面测试。</p>
* 开发及运维成本有所增加
<p class="li-explanation">&emsp;&emsp;微服务架构下可能要构建、开发、测试、部署、运行数十个甚至上百个独立的服务，并可能需要支持多种语言和环境，而且开发人员必须要有DevOps开发运维一体化的技能，这些都可能带来成本的增加。</p>
* 代码冗余
<p class="li-explanation">&emsp;&emsp;某些底层功能需要被多个服务所用，为了避免“同步耦合引入到系统中”，有时需要向不同的服务添加一些代码，这就会导致代码重复。</p>
* 服务间的协调问题
<p class="li-explanation">&emsp;&emsp;把系统分为多个服务组件之后，会产生新的接口，这意味着简单的交叉变化可能需要改变很多组件，比如，假设在一个案例中，需要修改服务A、B、C，而A依赖B，B依赖C，微服务架构模式就需要考虑相关改变对不同服务的影响，就需要更新服务C，然后是B，最后才是A。在实际应用环境中，一个新品发布可能被迫同时发布大量服务，由于集成点的大量增加，微服务架构会有更高的发布风险。</p>

任何事情都有两面性，微服务架构也会有其优缺点，是否需要使用微服务架构，还是要根据项目本身的需求进行考虑。

<br/>
注解：
<a name="IPC-explanation" href="#IPC">IPC：</a>Inter-Process Communication - 进程间通信
<a name="SOA-explanation" href="#SOA">SOA：</a>Service-Oriented Architecture - 面向服务的体系结构
<a name="ESB-explanation" href="#ESB">ESB：</a>Enterprise Service Bus - 企业服务总线
<a name="BPM-explanation" href="#BPM">BPM：</a>Business Process Management - 业务流程管理

<br/>
参考链接：
[解析微服务架构(一)：什么是微服务](https://www.ibm.com/developerworks/community/blogs/3302cc3b-074e-44da-90b1-5055f1dc0d9c/entry/%E8%A7%A3%E6%9E%90%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84_%E4%B8%80_%E4%BB%80%E4%B9%88%E6%98%AF%E5%BE%AE%E6%9C%8D%E5%8A%A1?lang=en)
[微服务实战（一）：微服务架构的优势与不足](http://dockone.io/article/394)