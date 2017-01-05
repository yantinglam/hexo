---
title: Spring Cloud - 服务注册与消费
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
&emsp;&emsp;Spring Cloud有众多子项目，如配置管理工具包Spring Cloud Config，可以把配置放在远程服务器上，集中化管理集群配置；又比如Spring Cloud Netfilx，该项目整合了Netflix OSS，可以直接在项目中使用Netflix OSS。想了解更多的Spring Cloud子项目，可以浏览https://springcloud.cc/ 。

### 服务注册中心
&emsp;&emsp;在之前的文章《[服务发现机制](/2016/12/15/Service-discovery/index.html)》中已经知道了服务发现机制对微服务架构的重要性。Spring Cloud框架实现服务发现是使用Eureka模块，是一种客户端发现机制。Netflix用Eureka模块来实现服务发现的服务端与客户端。
     
&emsp;&emsp;在Eureka中，服务端是一个“服务注册中心”，服务实例在启动时把自己的元数据诸如主机和端口等信息提供给注册中心，并且通过发送心跳包来维持注册状态。如果心跳包接收超时，实例将会从注册中心中移除。注册中心主要是处理请求的接收者。
     
&emsp;&emsp;某个服务实例需要访问其他的服务实例，需要在注册中心上，通过负载均衡机制找到相对应服务的网络位置，然后进行访问。

#### 创建服务注册中心
1.创建一个基础Spring Boot工程（在https://start.spring.io/ 中创建下载再导入到IDE中），并在pom.xml引入Eureka Server的依赖。
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
            <version>Brixton.SR5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

2.在application.properties文件中进行配置。
``` 
server.port=1111
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```

* server.port：指定注册中心的访问端口
* eureka.client.register-with-eureka：是否注册到注册中心上，注册中心本身就无需注册了。该配置默认为true
* eureka.client.fetch-registry：该服务是否被获取到，同样注册中心本身无需被获取。该配置默认为true
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
这里还有一个@SpringBootApplication注解，此注解等价于@Configuration、@EnableAutoConfiguration以及@ComponentScan的集合，用于配置Spring程序。

4.启动程序，访问http://localhost:1111/ ，可以看到服务注册中心的管理界面

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Eureka-Server.png" width=600 height=400 />

&emsp;&emsp;在这个界面上可以看到注册中心的状态以及服务的注册信息，通过“Instances currently registered width Eureka”这一栏可以知道现在还没有任何服务实例注册在上面。

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/None-instances-registered.png" width=800 height=500 />

### 注册服务到注册中心
&emsp;&emsp;现在尝试创建一个简单功能的客户端服务程序，例如提供计算简单a+b功能的服务程序，并把该服务注册到注册中心上。

1.创建一个简单的Spring Boot工程，并在pom.xml引入Eureka客户端依赖。
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
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
spring.application.name=compute-service
server.port=2222
eureka.client.serviceUrl.defaultZone = http://localhost:1111/eureka/
```
* spring.application.name：配置服务实例的名字，在后续的调用中，可以直接通过该名字对此服务进行访问
* server.port：指定服务实例的访问端口
* eureka.client.serviceUrl.defaultZone：指定要注册到上面的服务注册中心的位置

3.向入口程序添加@EnableDiscoveryClient注解，激活DiscoveryClient的实现，这样才能注册到注册中心上面。

``` java
@EnableDiscoveryClient //该注释能激活DiscoveryClient的实现，实现controller中的信息输出
@SpringBootApplication
public class ComputeServiceApplication {
	public static void main(String[] args) {
		new SpringApplicationBuilder(ComputeServiceApplication.class).web(true).run(args);
	}
}
```

4.编写处理请求程序
新建一个类，如ComputeController.java，一般会放在web目录下。

``` java
@RestController
public class ComputeController {

    private final Logger logger = Logger.getLogger(getClass());

    @Autowired
    private DiscoveryClient client;

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add(@RequestParam Integer a, @RequestParam Integer b) {
        Integer r = a + b;

        ServiceInstance instance = client.getLocalServiceInstance();
        logger.info("a: " + a + " b: " + b);
        logger.info("/add, host: " + instance.getHost() + ", service_id: " + instance.getServiceId() + ", result: " + r);

        return r;
    }
}
```

&emsp;&emsp;这里需要用到三个注解：@RestController、@RequestMapping以及@Autowired。
* @RestController注解被称为构造型注解，表明该类是一个支持REST的控制器。该注解告诉Spring以字符串的形式渲染结果，并直接返回给调用者
* @RequestMapping注解提供路由信息，在这个例子中，它告诉Spring将来自"/add"路径的HTTP请求都映射到add()方法，请求使用GET方法
* @Autowired注解可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作，以此来消除get、set方法

5.启动程序
&emsp;&emsp;访问http://localhost:2222/add?a=1&b=2 ，可以得到结果3，说明成功访问到该服务。此时刷新http://localhost:1111/ ,可以看到该服务已经注册到了注册中心上：

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Compute-service-registered.png" width=800 height=500 />

### 消费服务实例
&emsp;&emsp;现在有了注册中心，也有了一个能提供服务的实例注册到了注册中心上面，那么如何来消费这个服务实例呢？接下来再新建一个基础的Spring Boot工程，用这个工程来消费上面创建的compute-service服务。

&emsp;&emsp;在编写这个消费程序之前，要知道Netflix里的这个Eureka模块是服务发现机制里的客户端发现模式，而客户端发现模式在请求一个服务时，会到服务注册中心中查询，然后使用负载均衡算法从多个服务实例中选择一个，再发出请求。这部分内容可以回顾《[服务发现机制](/2016/12/15/Service-discovery/index.html)》。所以负载均衡在这里也是一个很重要的知识点。

&emsp;&emsp;在Netflix中，使用Ribbon模块来实现负载均衡。Ribbon是一个基于HTTP和TCP客户端的负载均衡，可以通过客户端中配置的ribbonServerList服务端列表去轮询访问以达到负载均衡的作用。

&emsp;&emsp;当Ribbon与Eureka配合使用时，ribbonSeverList会被DiscoveryEnableNIWSSeverList重写，这时就会变成从Eureka注册中心中获取服务端列表，同时也会用NIWSDiscoveryPing来取代IPing，它将职责委托给Eureka来确定服务端是否已经启动。

#### 使用Riboon实现客户端负载均衡的消费者
1.在刚刚新建的基础Spring Boot工程的pom.xml引入所需的Eureka客户端和Ribbon依赖。
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
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
spring.application.name=ribbon-consumer
server.port=3333
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
* spring.application.name：配置服务实例的名字，在后续的调用中，可以直接通过该名字对此服务进行访问
* server.port：指定服务实例的访问端口
* eureka.client.serviceUrl.defaultZone：指定要注册到上面的服务注册中心的位置

消费服务程序也需要注册到注册中心上，因为只有注册到上面，才能通过注册中心查找其他服务的位置进行消费。

3.向入口程序添加@EnableDiscoveryClient

``` java
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonApplication {

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
* @EnableDiscoveryClient注解激活DiscoveryClient实现，成为服务客户端，注册到服务注册中心上面
* @LoadBalanced注解开启负载均衡的能力
* @Bean注解声明一个Bean，是xml中的<bean/>元素的直接模拟
* RestTemplate类提供HTTP的各类方法，如GET、POST、HEAD、PUT、DELETE等，用于访问REST接口

4.创建ComputeService类来消费compute-service服务
新建一个类，如ComputeService.java，一般放在web目录下。

``` java
@Service
public class ComputeService {

    @Autowired
    RestTemplate restTemplate;

    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
}
```
* @Service注解用于标注业务组件
* ResponseEntity<T> getForEntity(String url, Class<T> responseType)方法的作用是，向url发起GET请求，返回的结果被转换成responseType类型并保存到ResponseEntity对象里。在此例子中，就是对http://COMPUTE-SERVICE/add?a=10&b=20 这个URL发起GET请求，此URL就是指向compute-service服务的，返回的结果被转换成String类型保存到了ResponseEntity对象中

5.编写请求处理程序
新建一个类，如ConsumerController.java，一般放在web目录下。

``` java
@RestController
public class ConsumerController {
    @Autowired
    private ComputeService computeService;

    @RequestMapping(value="/add", method= RequestMethod.GET)
    public String add() {
        return computeService.addService();
    }
}
```

6.启动程序
此时在注册中心上可以看到消费服务也注册到上面了，此时运行http://localhost:3333/add ,可以看到通过compute-service计算，返回来的结果30。

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Ribbon-registered.png" width=800 height=500 />

&emsp;&emsp;因为现在只启动了一个compute-service服务实例，所以看不出负载均衡的效果，可以将compute-service项目打包成jar包，然后通过命令行启动两个或多个compute-service服务实例。

&emsp;&emsp;注意：打包的时候，每个compute-service服务的server.port端口要设置成不一样。

&emsp;&emsp;IDEA打包jar可以参考此链接：http://blog.csdn.net/tolcf/article/details/50676171

&emsp;&emsp;打包成功后，依次启动注册中心服务、compute-service服务以及消费服务，此时在注册中心管理页面中看到有两个使用不同端口的compute-service服务注册到了上面：

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Two-compute-service.png" wdith=800 height=500 />

&emsp;&emsp;此时访问两次http://localhost:3333/add ，会看到两个compute-service都增加了一条log信息，说明两个服务都被访问了一次

端口为2222的服务实例：

<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Compute-service1.png" width=800 height=500 />

端口为2223的服务实例：
<img src="/images/SpringCloud/Spring-Cloud-service-discovery/Compute-service2.png" width=800 height=500 />

<br/>
参考链接：
[Spring Cloud构建微服务架构（一）服务注册与发现](http://blog.didispace.com/springcloud1/)
[Spring Cloud构建微服务架构（二）服务消费者](http://blog.didispace.com/springcloud2/)