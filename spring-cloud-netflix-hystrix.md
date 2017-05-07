# A Guide to Spring Cloud Netflix – Hystrix  

## 1. Overview 概览

这篇教程将会讲解Spring Cloud Netflix Hystrix，Spring Cloud Netflix Hystrix是一个容错库，我们将会使用这个库来实现断路器企业级模式，这个模式描述了一种策略来防止service层的不同级别的级联失败。  

> **简单的说，怎样允许一个service继续工作，当它调用的service失败的时候？**

这个原理和电子学类似。Hystrix监控着相关服务的方法是否有失败的调用。如果发现了一个失败的方法，它就会断开电路。意思是，它将这个失败的调用转向一个fallback 后备方法。  

这个库会允许错误达到一个临界值。超过临界值，它将会让电路永远断开。意思是，它将会把随后的所有调用转向到fallback方法，来避免以后的错误。这会为相关的服务从失败状态恢复创建一个时间缓冲。  

## 2. REST Producer REST 生产者  

为了创建一个情景，来演示断路器模式，我们首先需要一个相关的service.我们给它命名为“REST Producer”.因为它为开启了Hystrix的“REST Consumer”提供了数据,下一步我们将会创建这个消费者。    

如果你已经熟悉了REST service的实现。或者你已经有了一个=基本的REST service框架，你可以跳过这个小节。  

如果不是这样，我们创建一个新的Maven项目，使用[spring-boot-starter-web](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)依赖。 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
```

这个项目有意的保持简单。项目将会由一个controller interface并带有一个@RequestMapper注解的GET方法，简单的返回一个String，和一个 @RestController 实现了前边的接口，还有一个@SpringBootApplication.   

We’ll begin with the interface:  

```
public interface GreetingController {
    @RequestMapping("/greeting/{username}")
    String greeting(@PathVariable("username") String username);
}
```
And the implementation:  

```
@RestController
public class GreetingControllerImpl implements GreetingController {
    @Override
    public String greeting(@PathVariable("username") String username) {
        return String.format("Hello %s!\n", username);
    }
}
```  

Next we’ll write down the main application class: 

```
@SpringBootApplication
public class RestProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestProducerApplication.class, args);
    }
}
```  

为完成这一小节，唯一剩下要做的是，配置应用端口，我们将不会使用8080，因为这个端口应该留给下一步要做的应用。  

此外我们定义了一个应用名称，为了client application能够查找我们的"REST Producer",我们将稍后介绍client application.  

Let’s create an application.properties with the following content: 

```
server.port=9090
spring.application.name=rest-producer
```
Now we’re able to test our ‘REST Producer’ using curl:  

```
$> curl http://localhost:9090/greeting/Cid
Hello Cid!
```  

## 3. REST Consumer with Hystrix REST消费者使用 Hystrix

为了要演示的情节，我们将实现一个web应用，web应用负责消费上个小节中创建的REST service，使用RestTemplate and Hystrix，为了简单起见，我们称它为“REST Consumer”。  

因此，我们创建一个新的Maven项目，使用[spring-cloud-starter-hystrix](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-hystrix%22) [spring-boot-starter-web](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) and [spring-boot-starter-thymeleaf](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-thymeleaf%22) 作为依赖：  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>1.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
```  

为了断路器能工作，Hystix 将会扫描 @Component or @Service 注解的类为找到 @HystixCommand注解的方法，为该方法实现一个代理，并且监视它的调用。  

我们将要首先创建一个@Service 类，service类会被autowired到@Controller里边。既然我们正在构建一个[a web-application using Thymeleaf](http://www.baeldung.com/thymeleaf-in-spring-mvc).我们还需要一个HTML模板作为试图。  

这将会是可注入的@Service通过一个关联的fallback方法实现一个@HystrixCommand。这个fallback必须使用相同的签名（参数类型，参数名，返回值类型）和原始监听的方法。  


```
@Service
public class GreetingService {
    @HystrixCommand(fallbackMethod = "defaultGreeting")
    public String getGreeting(String username) {
        return new RestTemplate()
          .getForObject("http://localhost:9090/greeting/{username}", 
          String.class, username);
    }
 
    private String defaultGreeting(String username) {
        return "Hello User!";
    }
}
```

RestConsumerApplication 将会是我们的主application类，这里的@EnableCircuitBreaker注解将会扫描classpath为找到兼容的断路器实现。  

To use Hystrix explicitly you have to annotate this class with @EnableHystrix:  

为了显式的使用Hystrix，我们必须使用@EnableHystrix注解这个类：  

```
@SpringBootApplication
@EnableCircuitBreaker
public class RestConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestConsumerApplication.class, args);
    }
}
```  

We’ll set up the controller using our GreetingService:

```
@Controller
public class GreetingController {
    @Autowired
    private GreetingService greetingService;
 
    @RequestMapping("/get-greeting/{username}")
    public String getGreeting(Model model, @PathVariable("username") String username) {
        model.addAttribute("greeting", greetingService.getGreeting(username));
        return "greeting-view";
    }
}
```
And here’s the HTML template:  

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Greetings from Hystrix</title>
    </head>
    <body>
        <h2 th:text="${greeting}"/>
    </body>
</html>
```  

为确保这个应用监听的是默认的端口，we put the following in an application.properties file:  

```
server.port=8080
```  

为看到Hystrix断路器在运转，我们启动“REST Consumer”并且浏览器打开http://localhost:8080/get-greeting/Cid。在正常情况下，the following will be shown:  

```
Hello Cid!
``` 

为模拟“REST Producer”的失败，我们将简单的停掉它，然后刷新浏览器，我们应该看到一般的信息，从@Service中的fallback方法中返回。  

```
Hello User! 
```

## 4. REST Consumer with Hystrix and Feign REST消费者使用Hystrix和Feign REST   


现在我们将要修改上一节中的项目，来使用Spring Netflix Feign，作为声明式的REST client,而不是 Spring RestTemplate.  

这里的好处是，我们在之后将能够简单地重构Feign Client Interface来使用[Spring Netflix Eureka](http://www.baeldung.com/spring-cloud-netflix-eureka)为服务发现。  

为了开始这个新的项目，我们将复制一份“REST Consumer”并且把“REST Producer”和spring-cloud-starter-feign作为依赖加入到pom.xml中：  

```
<dependency>
    <groupId>com.baeldung.spring.cloud</groupId>
    <artifactId>spring-cloud-hystrix-rest-producer</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.1.5.RELEASE</version>
</dependency>
```  

现在，我们能够使用GreetingController来拓展Feign Client.我们将实现Hystrix fallback作为内部静态类并使用@Component注解。  

另一种可选方法是，我们可以定义一个@Bean注解的方法返回这个fallback类的实例。  

@FeignClinet中的name属性是强制的。它被用来寻找指定的应用，可以是通过服务发现的 Eureka Client也可以是url,如果这个属性已经给出来了。了解更多的使用Spring Netflix Eureka for 服务发现看[this article](http://www.baeldung.com/spring-cloud-netflix-eureka):  

```
@FeignClient(
  name = "rest-producer"
  url = "http://localhost:9090", 
  fallback = GreetingClient.GreetingClientFallback.class
)
public interface GreetingClient extends GreetingController {
     
    @Component
    public static class GreetingClientFallback implements GreetingController {
        @Override
        public String greeting(@PathVariable("username") String username) {
            return "Hello User!";
        }
    }
}
```

在RestConsumerFeignApplication中，我们将加入附加的注解来开启Fiegn 集成。实际上，是@EnableFeignClients这个注解。

```
@SpringBootApplication
@EnableCircuitBreaker
@EnableFeignClients
public class RestConsumerFeignApplication {
     
    public static void main(String[] args) {
        SpringApplication.run(RestConsumerFeignApplication.class, args);
    }
}
```  
我们将要修改这个controller来使用auto-wired Feign Client而不是之前的注入的Service.来重新获取greeting：

```
@Controller
public class GreetingController {
    @Autowired
    private GreetingClient greetingClient;
 
    @RequestMapping("/get-greeting/{username}")
    public String getGreeting(Model model, @PathVariable("username") String username) {
        model.addAttribute("greeting", greetingClient.greeting(username));
        return "greeting-view";
    }
}
```  

为把这个项目跟上一个项目区别开，我们将会修改应用的监听端口，in the application.properties:  

```
server.port=8082
```  
最后我们测试这个Feign-enabled 'REST Consumer'像上个小节中的一样。预期结构也应该一样。  

## 5. Using Scopes （使用范围，监听方法的运行范围scope） 

一般情况下，一个@HytrixCommand注解的方法，会被在一个线程池中context中执行。但是有些时候它需要运行在本地范围 local scope，比如，@SessionScope or a @RequestScope。这个可以被实现通过在command注解中加几个参数：   

```
@HystrixCommand(fallbackMethod = "getSomeDefault", commandProperties = {
  @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
})
```  

## 6. The Hystrix Dashboard 仪表盘 

一个可选的Hystrix很好的功能是在仪表盘监视断路器的状态。  
为开启这个功能，spring-cloud-starter-hystrix-dashboard and spring-boot-starter-actuator in the pom.xml of our ‘REST Consumer’:  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    <version>1.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
```  
前者需要被开启通过在@Configuration加上@EnableHystrixDashboard注解，后一个会在应用中自动开启必须的指标参数。  

重启应用之后，浏览器打开http://localhost:8080/hystrix,input the metrics url of a ‘hystrix.stream’ and begin monitoring. Finally we should see something like this:  

![Hystrix Dashboard](http://inprogress.baeldung.com/wp-content/uploads/2016/08/Screenshot_20160819_031730-268x300.png)  

监视一个hystrix.stream’是很好，但是如果你必须监视多个开启了Hystrix的应用，这就会变得不方便。为实现这个目的，Spring Cloud提供了一个工具叫作Turbine,这个工具能够聚合多个streams来呈现到一个Hystrix仪表盘上。  

配置Turbine已经超出了本位的范畴，但是这个实现的可能应该在此处被提及。所以，通过信息收集这些streams是可能的。  

## 7. Conclusion 总结  

到目前我们看到的，我们能够实现断路器模式通过Spring Netflix Hystrix和Spring RestTemplate的组合或者和Spring Netflix Feign组合。  

This means that we’re able to consume services with included fallback using ‘static’ or rather ‘default’ data and we’re able to monitor the usage of this data.  

这意味着我们能够消费服务的时候使用fallback，通过使用'static'或者默认数据实现，并且我们能够监视这些数据的使用。


 





