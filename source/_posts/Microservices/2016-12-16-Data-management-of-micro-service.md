---
title: 微服务的数据管理问题
date: 2016-12-16 11:00:05
categories:
- 技术
tags:
- 微服务架构
---
### 数据管理情况
&emsp;&emsp;在微服务架构中，数据是服务私有的，外部要想访问某个服务的数据，唯一的访问方式是通过API。如果多个服务访问同一个数据，schema会更新访问时间，并在所有服务之间进行协调。也会有不同的微服务使用不同的数据库的情况，在这种情况下，应用会产生各种不同的数据，因此基于微服务的应用一般都使用SQL和NoSQL结合的数据库。
<!-- more -->
### 数据管理面临的挑战
* 如何完成一笔交易的同时保持多个服务之间的数据一致性
* 如何完成从多个服务中搜索数据，因为不同的服务之间数据库的设计不一样，所设置的查询条件也不一样

### 事件驱动架构（event driven architecture）
&emsp;&emsp;事件驱动架构是使用事件来实现跨多服务的业务交易。

#### 事件驱动的实现机制
&emsp;&emsp;当某件重要的性事情发生时，微服务会发布一个事件，如更新一个业务实体事件，当订阅这个事件的微服务接收到此事件时，就可以消费这个事件来更新自己的业务实体，如果有需要，这个服务会继续发出下一个事件，以此循环，直到所有事件处理完毕。

&emsp;&emsp;基于这种交易模式的被称作为BASE model，它提供的是一种弱确定性，如最终一致性。
<img src="/images/Microservices/Data-management-of-micro-service/Event-driven1.png" width=500 height=500 />
<img src="/images/Microservices/Data-management-of-micro-service/Event-driven2.png" width=500 height=500 />
<img src="/images/Microservices/Data-management-of-micro-service/Event-driven3.png" width=500 height=500 />
#### 事件驱动的优点
* 可以使得交易跨多个服务且提供最终一致性
* 可以使应用维护最终视图
<img src="/images/Microservices/Data-management-of-micro-service/Event-driven4.png" width=500 height=500 />

#### 事件驱动的缺点：
* 编程模式比ACID更加复杂 
* 为了从应用层级失效中恢复，还需要完成补偿性交易
* 当应用读取未更新的最终视图时，也会出现数据不一致问题
* 事件订阅者必须检测和忽略冗余事件

### 处理原子性操作问题
&emsp;&emsp;数据库更新和事件的发布是一个原子操作.
#### 获得原子性的方法：
##### 使用本地交易发布事件
&emsp;&emsp;对发布事件应用采用multi-step process involving only local transactions，关键在于一张EVENT表，此表在存储业务实体数据库中起到消息列表的功能。
###### 实现机制
&emsp;&emsp;应用发起一个（本地）数据库交易，如更新业务实体状态，先向EVENT表中插入一个事件，然后提交此次交易；另外一个独立的应用进程或线程查询此EVENT表，向消息代理发布事件，然后使用本地交易标志此事件为已发布。
<img src="/images/Microservices/Data-management-of-micro-service/Local-event.png" width=500 height=500 />
###### 使用本地交易发布事件的优点
* 可以确保事件发布不依赖于<a name="2PC" href="#2PC-explanation">2PC</a>
* 应用发布业务层级事件而不需要推断他们发生了什么

###### 使用本地交易发布事件的缺点
* 开发人员必须牢记发布事件，因此有可能出现错误
* 对某些使用NoSQL数据库的应用是个挑战，因为NoSQL本身的交易和查询能力有限

##### 挖掘数据库交易日志
&emsp;&emsp;此方法是应用更新数据库，使数据库交易日志发生了变化，交易日志挖掘进程或线程读这些交易日志，将日志发布给消息代理。
<img src="/images/Microservices/Data-management-of-micro-service/Dig-DB.png" width=500 height=500 />
###### 挖掘数据库交易日志优点
* 确保每次更新发布事件不依赖于2PC
* 交易日志挖掘可以通过将发布事件和应用业务逻辑分离得到简化

###### 挖掘数据库交易日志缺点
* 交易日志对不同的数据库有不同的格式，甚至不同数据库版本也有不同的格式
* 很难从底层交易日志更新记录转换为高层业务事件

##### 使用事件源
&emsp;&emsp;此方法的应用保存业务实体一系列状态改变的事件，而不是存储实体现在的状态，应用可以通过重放事件来重建实体现在的状态。只要业务实体发生改变，新事件就会添加到时间表中，因为保存事件是单一操作，所以肯定是原子性的操作。如在事件源方式中，订单服务以事件状态改变方式存储一个订单：创建的、已批准的、已发货的、取消的；每个事件包含足够的信息来重建订单状态。

&emsp;&emsp;事件存储是事件驱动微服务架构的基干，它跟消息代理类似，将事件递送到所有感兴趣的订阅者，订阅都通过提供的API来订阅事件。事件被长期保存在事件数据库中，通过API来添加或获取实体事件。
<img src="/images/Microservices/Data-management-of-micro-service/Event-source.png" width=700 height=700 />
###### 使用事件源的优点：
* 解决事件驱动架构关键问题，使得只要有状态变化就可以可靠地发布事件，也就解决了微服务架构的数据一致性问题
* 因为是持久化事件而不是对象，就不会出现object relational impedance mismatch problem（对象-关系阻抗失配问题）
* 提供了100%可靠的业务实体变化监控日志，使用获取任何时点的实体状态成为可能
* 使得业务逻辑可以由事件交换的松耦合业务实体构成
* 使得单体应用移值到微服务时变得相对简单

###### 使用事件源的缺点：
* 事件存储只支持主键查询业务实体
* 必须使用Command Responsiblity Segregation(CQRS)来完成查询业务，因此应用必须处理最终一致数据
* 因为采用不同或不太熟悉的变成模式，重新学习不太容易 

<br/>
注释：
<a name="2PC-explanation" href="#2PC">2PC：</a>Tow-phase Commit Protocol - 二阶段提交协议

<br/>
参考链接：
[微服务实践（五）：微服务的事件驱动数据管理](http://dockone.io/article/936)