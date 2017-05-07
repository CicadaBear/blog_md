# Introduction to Spring Cloud Rest Client with Netflix Ribbon 

From [Baeldung](http://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon)

## 1. Introduction 简介  

Netflix [Ribbon](https://github.com/Netflix/ribbon) 是一个IPC cloud library. Ribbon 主要功能是提供了client-side 负载均衡算法。  

除了 client-side 负载均衡之外，Ribbon也提供了其他功能。

* Service Discovery Integration（跟服务发现集成） - Ribbon负载均衡提供了动态环境中的服务发现。与Eureka和Netflix服务发现集成组件已经包含在ribbon的lib中了。 
* Fault Tolerance（容错）- Ribbon Api 可以动态的决定，是否一个服务是开启的并且运行在一个存活的环境中，还可以探测停止运行的服务。  
* Configurable load-balancing rules（可配置的负载均衡规则）- Ribbon支持 RoundRobinRule,AvailabilityFilteringRule,WeightedResponseTimeRule,3个负载均衡规则，并且支持自定义规则。  

Ribbon Api的工作基于一个叫作“Named Client”的概念。While configuring Ribbon in our application configuration file we provide a name for the list of servers included for the load balancing.当在我们的应用中配置Ribbon时，我们为the list of servers included for the load balancing提供一个名字。  

Let’s take it for a spin.

## 2. Dependency Management 依赖管理  

The Netflix Ribbon API can be added to our project by adding the below dependency to our pom.xml: 

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

## 3. Example Application 示例应用  

为了看Ribbon Api的工作，我们用RestTemplate构建了一个简单的微服务应用，我们用Ribbon Api增强和Spring Cloud一起工作。  

我们将会使用Ribbon的负载均衡策略之一WeightedResponseTimeRule, 来开启这个client side负载均衡在两个服务之间，这两个服务在哪里定义的，在application配置文件named client下边。  

## 4. Ribbon Configuration Ribbon 配置  

Ribbon API 使我们能够配置以下几个负载均衡的组件。  

* Rule 规则 - 指定负载均衡规则的逻辑组件
* Ping 状态监测 - 我们用来指定实时监测服务可用性的组件  
* ServerList 服务列表 - 动态静态都可以。在我们的例子中，我们使用了一个静态的服务列表并且因此我们直接将他们定义在application的配置文件中。  

Let write a simple configuration for the library:  

```
public class RibbonConfiguration {
 
    @Autowired
    IClientConfig ribbonClientConfig;
 
    @Bean
    public IPing ribbonPing(IClientConfig config) {
        return new PingUrl();
    }
 
    @Bean
    public IRule ribbonRule(IClientConfig config) {
        return new WeightedResponseTimeRule();
    }
}
```  

需要注意的是，我们是怎样使用WeightedResponseTimeRule规则的和PingUrl机制来决定服务的实时可用性的。  

关于这个规则，每一个服务被设定一个weight权重关于它的的平均响应时间，lesser the response time gives lesser the weight，响应时间越短，权重越小（是不是有问题，自己知道就行了）。这个规则随机选择一个服务，这个服务被选中的可能性取决于它的权重。   

PingUrl 将会ping 每一个url，来决定这个服务的可用性。  

## 5. application.yml  

Below is the application.yml configuration file we created for this sample application:  

```
spring:
  application:
    name: spring-cloud-ribbon
 
server:
  port: 8888
 
ping-server:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:9092,localhost:9999
    ServerListRefreshInterval: 15000

```  

在上边的配置文件中，我们指定了：  

* Application name 
* 应用端口号
* Named client for the list of servers: “ping-server” 服务列表的Named client 名称 ping-server
* 禁用了Eureka服务发现组件，通过设置 eureka:enbaled false
* 为负载均衡定义了一个服务列表，在这个例子中，给两个服务做负载均衡。
* 配置服务刷新速率，ServerListRefreshInterval  

## 6. RibbonClient 

Let’s now set up the main application component snippet – where we use the RibbonClient to enable the load balancing instead of the plain RestTemplate:  

现在来设置主应用组件片段，我们在这个片段中使用RibbonClient来开启负载均衡，取代直接使用普通的RestTemplate。

```
@SpringBootApplication
@RestController
@RibbonClient(
  name = "ping-a-server",
  configuration = RibbonConfiguration.class)
public class ServerLocationApp {
 
    @LoadBalanced
    @Bean
    RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
 
    @Autowired
    RestTemplate restTemplate;
 
    @RequestMapping("/server-location")
    public String serverLocation() {
        return this.restTemplate.getForObject(
          "http://ping-server/locaus", String.class);
    }
 
    public static void main(String[] args) {
        SpringApplication.run(ServerLocationApp.class, args);
    }
}
```
我们定义了一个RestController.我们也加上了@RibbonClient的注解，并且配置了 name 和 configuration class。
 
我们在这定义的这个配置类，和我们之前定义的，我们提供了想要的RibbonAPI 配置在那个类中，这两个配置类是同一个。  

注意，我们也为RestTemplate加上了@LoadBalanced的注解。这个注解的含义是，我们想要负载均衡在这个例子中是用Ribbon负载均衡。  

## 7. Failure Resiliency in Ribbon      Ribbon中的错误弹性  

正如我们在这篇文章前边探讨过的，Ribbon API 不仅仅提供client side负载均衡，它还建立了错误弹性。  

在开始之前，Ribbon API 可以决定一个服务的可用性通过持续的以一定间隔的ping，并且可以跳过那些不在活跃状态的服务。  

附加在上边的还有，它还实现了断路器模式来过滤服务基于一个指定的标准。  

这个断路器模式最小化了一个服务失败所造成的影响，通过快速的拒绝请求到那个失败的服务，而不是等待超时。我们可以禁用断路器功能通过设置property niws.loadbalancer.availabilityFilteringRule.filterCircuitTripped to false.  

当所有的服务都down了，因此没有服务可用来处理请求，pingUrl() 将会失败并且接受一个java.lang.IllegalStateException异常，并带有错误信息 “No instances are available to serve the request”.  

## 8. Conclusion 总结  

在这篇文章中，我们探讨了Netflix Ribbon API，还有在一个简单的演示应用中它的实现。  

The complete source code for the example described above can be found on the [GitHub repository](https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-ribbon-client).





