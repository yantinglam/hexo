---
title: Spring Cloud - Feign
date: 2016-12-28 17:28:33
categories:
- 技术
tags:
- 微服务
- Spring Cloud
---
### 什么是Feign？
&emsp;&emsp;Feign是一个声明式的Web Service客户端，它旨在通过最少的资源和代码来实现和HTTP API的连接，使得编写Web Service客户端变得更加简单。它的使用方法是定义一个接口，然后在上面添加注解（支持JAX-RS标准的注解)。Feign支持可插拔式的编码器和解码器。
<!-- more -->
&emsp;&emsp;Spring Cloud对Feign进行了封装，使其支持Spring MVC标准注解和HttpMessageConverters，还整合了Ribbon和Eureka来提供负载均衡的HTTP客户端实现。

### 创建一个Spring Cloud Feign工程
1.先新建一个基础的Spring Boot工程，并通过配置pom.xml文件引入Feign和Eureka模块相关依赖。
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Brixton.SR5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
2.在application.properties文件中进行配置。
``` properties
spring.application.name=feign-consumer
server.port=4444
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```

* spring.application.name：配置服务实例的名字，在后续的调用中，可以直接通过该名字对此服务进行访问
* server.port：指定服务实例的访问端口
* eureka.client.serviceUrl.defaultZone：指定要注册到上面的服务注册中心的位置

3.向入口程序添加@EnableDiscoveryClient以及@EnableFeignClients注解

``` java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(FeignApplication.class, args);
	}
}
```

* @EnableDiscoveryClient注解开启DiscoveryClient功能
* @EnableFeignClients注解开启Feign功能

3.定义compute-service服务的接口
新建一个接口，如ComputeClient.java

``` java 
@FeignClient(value = "compute-service")  
public interface ComputeClient {
    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```
* @FeignClient注解绑定该接口对应compute-service服务
* 通过Spring MVC注解来配置compute-service服务下的具体实现。如@RequestMapping注解处理路由信息，@RequestParam注解绑定请求参数

4.编写处理请求程序
新建一个类，如ConsumerController.java，一般放在web目录下。

``` java
@RestController
public class ConsumerController {
    @Autowired
    ComputeClient computeClient; 

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add() {
        return computeClient.add(10, 20);
    }
}
```

* 因为ComputeClient接口绑定了compute-service服务，所以调用ComputeClient的add方法就相当于调用compute-service的add方法。

5.依次启动注册中心服务，compute-service服务（参考[Spring Cloud - 服务注册与消费](/2016/12/20/Spring-Cloud-service-discovery/index.html)）以及此Feign客户端服务，在注册中心管理页面上看到已注册到上面的服务，此时访问两次http://localhost:4444/add ，能得到通过compute-servcie服务计算出的结果，而且看到两个compute-service服务实例都有被访问到。

<img src="/images/SpringCloud/Spring-Cloud-Feign/Feign-registered.png" width=800 heigth=500 />

端口为2222的服务实例：

<img src="/images/SpringCloud/Spring-Cloud-Feign/Compute-service1.png" width=800 height=500 />

端口为2223的服务实例：
<img src="/images/SpringCloud/Spring-Cloud-Feign/Compute-service2.png" width=800 height=500 />


