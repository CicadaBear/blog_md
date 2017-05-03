# Introduction to Spring Cloud Rest Client with Netflix Ribbon 

From [Baeldung](http://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon)

## 1. Introduction  

Netflix [Ribbon](https://github.com/Netflix/ribbon) 是一个IPC cloud library. Ribbon 主要功能是提供了client-side 负载均衡算法。  

除了 client-side 负载均衡之外，Ribbon也提供了其他功能。

* Service Discovery Integration（跟服务发现集成） - Ribbon负载均衡提供了动态环境中的服务发现。与Eureka和Netflix服务发现集成组件已经包含在ribbon的lib中了。 
* Fault Tolerance（容错）- Ribbon Api 可以动态的决定，是否一个服务是开启的并且运行在一个存活的环境中，还可以探测停止运行的服务。  
* Configurable load-balancing rules（可配置的负载均衡规则）- Ribbon支持 RoundRobinRule,AvailabilityFilteringRule,WeightedResponseTimeRule,3个负载均衡规则，并且支持自定义规则。  

Ribbon Api的工作基于一个叫作“Named Client”的概念。While configuring Ribbon in our application configuration file we provide a name for the list of servers included for the load balancing.当在我们的应用中配置Ribbon时，我们为the list of servers included for the load balancing提供一个名字。  

Let’s take it for a spin.

## 2. Dependency Management  

The Netflix Ribbon API can be added to our project by adding the below dependency to our pom.xml: 

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

## 3. Example Application  

为了看Ribbon Api的工作，我们用RestTemplate构建了一个简单的微服务应用，我们用Ribbon Api增强和Spring Cloud一起工作。

