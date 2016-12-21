---
title: Spring Cloud - 服务注册与发现
date: 2016-12-20 14:27:29
categories:
- 技术
tags:
- 微服务
- Spring Cloud
---
### 什么是Spring Cloud？
&emsp;&emsp;Spring Cloud是一个基于Spring Boot实现的云应用开发工具，它为基于JVM的云应用开发中的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。
<!-- more -->     
&emsp;&emsp;Spring Cloud有众多子项目，如配置管理工具包-Spring Cloud Config，可以把配置放在远程服务器上，集中化管理集群配置；又比如Spring Cloud Netfilx整合了Netflix OSS，可以直接在项目中使用Netflix OSS。想了解更多的Spring Cloud子项目，可以浏览https://springcloud.cc/

### 服务注册中心
&emsp;&emsp;在之前的文章[服务发现机制](/2016/12/15/Service-discovery/index.html)中已经知道了服务发现机制对微服务架构的重要性，Spring Cloud框架实现服务发现是使用Eureka模块，是一种客户端发现机制，Netflix用它来实现服务发现的服务端与客户端。
     
&emsp;&emsp;在Eureka中，服务端是一个“服务注册中心”，服务实例在启动时把自己的元数据诸如主机和端口、健康指标RUL以及首页等信息提供给注册中心，并且通过发送心跳包来维持注册状态。如果心跳包接收超时失败，实例将会从注册中心中移除。注册中心主要是处理请求的接收者。
     
&emsp;&emsp;某个服务实例需要访问其他的服务实例，需要在注册中心上，通过负载均衡机制找到相对应服务的网络位置，然后进行访问。

### 创建服务注册中心
1.创建一个基础Spring Boot工程，并在pom.xml引入Eureka Server的依赖。
``` xml
<dependencies>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
</dependencies>

<dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Brixton.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
</dependencies>
</dependencyManagement>
```

2.在application.properties文件中进行配置。
``` 
server.port = 1111
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```

* server.port：定义了服务注册中心的端口；
* eureka.client.register-with-eureka：是否注册到注册中心上，注册中心本身就无需注册了。该配置默认为true
* eureka.client.fetch-registry：该服务是否被获取到。该配置默认为true
* eureka.client.serviceUrl.defaultZone：指定服务注册中心的位置

3.向入口程序添加@EnableEurekaServer注解，开启服务注册中心功能。
``` java
@EnableEurekaServer
@SpringBootApplication
public class SpringCloudApplication {
	public static void main(String[] args) {
		new SpringApplicationBuilder(SpringCloudApplication.class).web(true).run(args);
	}
}
```
启动程序，访问http://localhost:1111/ ，可以看到服务注册中心的界面

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Eureka-Server.png" width=500 height=300 />

### 注册服务到注册中心
#### 创建客户端服务程序