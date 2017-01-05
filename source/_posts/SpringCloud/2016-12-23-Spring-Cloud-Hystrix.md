---
title: Spring-Cloud - 断路器
date: 2016-12-23 13:57:32 
categories:
- 技术
tags:
- 微服务
- Spring Cloud
---
&emsp;&emsp;在微服务架构中，是将一个个业务功能拆分成一个个小的服务，各个服务之间通过IPC机制进行通信，因此有可能因为网络故障或依赖服务自身问题出现延迟或调用故障。若此时调用方不断增加请求，最后就会出现因等待依赖服务响应而造成任务积压，最终导致自身服务的瘫痪。为了解决这个问题，Spring Cloud给出了断路器模式。
<!-- more -->
### 什么是断路器？
&emsp;&emsp;断路器的概念原本是应用在电路上的，断路器是一种开关装置，用于保护线路过载。当电路中有电器发生短路，断路器能及时切断故障电路，避免发生过载、过热甚至起火等严重后果。

&emsp;&emsp;在微服务架构中，断路器也起到了一个类似这样的作用。当某个服务发生故障时，通过断路器的故障监控，向调用方返回一个设定的错误响应，以防止调用方长期等待。这样可以避免线程因服务故障而被长期占用不能释放，从而避免故障蔓延。

### Spring Cloud的断路器实现
&emsp;&emsp;在Spring Cloud中，使用Hystrix来实现断路器的功能。Hystrix旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。

### Hystrix的实现
&emsp;&emsp;这里需要用到前面[服务注册与消费](/2016/12/20/Spring-Cloud-service-discovery/index.html)的例子。

&emsp;&emsp;依次启动注册中心服务，compute-service服务以及Ribbon客户端服务，并访问http://localhost:3333/add ,此时可以看到页面返回计算结果30。现在通过关闭compute-service来模拟服务出现故障的情况，关闭compute-service服务之后，再次访问http://localhost:3333/add ，会得到如下图所页的错误提示：

<img src="/images/SpringCloud/Spring-Cloud-Hystrix/Compute-service-error.png" width=500 height=300 />

&emsp;&emsp;接下来修改Ribbon客户端来增加Hystrix功能。
1.加入Hystrix依赖。

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

2.向主入口程序添加@EnableCircuitBreaker注解开启断路器功能。

``` java
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreakeraker //开启Hystrix
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

3.在服务消费方法上添加@HystrixCommand注解来指定失败回调方法

``` java 
@Service
public class ComputeService {

    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "addServiceFallback")
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }

    public String addServiceFallback() {
        return "error";
    }
}
```

* 此例子中指定失败回调方法为addServiceFallback()，该方法返回字符串"error" 

4.重启Ribbon服务，再次访问http://localhost:4444/add ，此时得到返回的错误字符串"error"。
<img src="/images/SpringCloud/Spring-Cloud-Hystrix/Compute-service-errorFallback.png" width=500 height=300 />

<br/>
参考链接：
[Spring Cloud构建微服务架构（三）断路器](http://blog.didispace.com/springcloud3/)