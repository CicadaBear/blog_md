#Spring Boot Bootstrapping

From [baeldung](http://www.baeldung.com/spring-cloud-bootstrapping)

这篇文章翻译自baeldung，这是个教程网站，有免费的有付费的，付费的多是视频，这个网站整个教程，文档水平还是非常之高的。之前关于spring-cloud的文章都是从spring的官网上翻译的，内容质量不能说不好，只是有点像本大部头的字典，非常之细，但对总体的把握不是很好。我是个贪多嚼不烂的人，兴趣广泛，不求甚解，只求我知道哪有本字典供我查询就可以了，所以baeldung的教程和Spring官方的文档相比，我自己认为baeldung的更好。之前的文章已经有一个月没有更新了，拖延，懒。我的博客也不全是对外，很大程度上是对自己的一个激励，一个里程碑，一个学习日志，很多前辈都说，当老师是学会一门知识的非常棒的方法。    

   
我会展示出baeldung的文档有多么的棒。   

##1. Overview 概览  


Spring Cloud 是个为构建强健的云应用的框架。当移动到分布式环境的时候，Spring Cloud对面对的常见问题提供解决方案，从而简化了应用的开发。       
    
微服务架构的应用的目标是简化开发、部署和维护。这种应用，已分解的本质允许开发者一次只关注一个问题。改进可以被引入，在不影响系统其他部分的情况下。   

在另一方面，但我们采用微服务方法的俄时候，不同的挑战也会出现。比如  

* 拓展配置，可以实现非常灵活，并且在改变的时候不必重建这个servie。
* 服务发现
* 隐藏部署在不同主机上的服务复杂的东西。

在这篇文章中，我们将要构建5个微服务，一个 configuration server(配置服务)，一个 discovery server (发现服务)，一个 gateway server(网关服务)，一个 book service(自定义的业务，图书服务)，一个 rating service (自定义的业务，评分服务)。这5个微服务从一个可靠的的基础应用变成云开发，并且处理前边提到的挑战。   

## 2. Config Server 配置服务   

当开发一个云应用的时候，一个问题是维护和部署配置到多个服务中。我们不想在水平拓展服务（scaling our service horizontally），或者遭受安全漏洞（risk security breaches），（通过baking our configuration into our application，调整配置解决），之前，配置每一个环境的时候花费时间。  

为解决这个问题，我们把所有的配置的合并到一个git仓库中，并且把git仓库和一个为所有应用管理配置的应用连接。我们将要设置一个简单的实现。   

To学习更多的细节并且看一个更复杂的示例，展开看另一篇文章只关于[Spring Cloud Configuration](http://www.baeldung.com/spring-cloud-configuration)的。   

### 2.1 Setup 设置  

Navigate to [start.spring.io](http://start.spring.io/) and select Maven and Spring Boot 1.4.x.   

使用 start.spring.io 的自动方法。  

Set the artifact to “config”. In the dependencies section, search for “config server” and add that module. Then press the generate button and you will be able to download a zip file with a preconfigured project inside and ready to go.


手动方法  
  
Alternatively, we can generate a Spring Boot project and add some dependencies to the POM file manually.

These dependencies will be shared between all the projects:


```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
    <relativePath/>
</parent>
 
<dependencies>
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
 
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

```

Let’s add a dependency for the config server:   

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

For reference: you can find the latest version on Maven Central ([spring-cloud-dependencies](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-dependencies%22), [test](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22), [config-server](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-config-server%22)).  

### 2.2. Spring Config  

To enable the configuration server we must add some annotations to the main application class:  

```
@SpringBootApplication
@EnableConfigServer
public class ConfigApplication {...}
```   
@EnableConfigServer will turn our application into a configuration server.   

### 2.3. Properties   

Let’s add the application.properties in src/main/resources:   

application.properties

```
server.port=8081
spring.application.name=config
 
spring.cloud.config.server.git.uri=file://${user.home}/application-config
```  
config server最重要的设置是git.url参数。这个参数最近设置成相对路径，一般情况下 Windows c:\Users\{username}\ 或者 Linux /Users/{username}/. 这个属性指向的是存储了所有配置文件的Git仓库。如果必要，也可以是一个绝对路径。 

Tip: On a windows machine preface the value with ‘file:///’, on *nix then use ‘file://’.  
  

### 2.4. Git Repository  

spring.cloud.config.server.git.uri 指向的已经是配置仓库了，文件夹下初始化git。

### 2.5. Run

Let’s run config server and make sure it is working. From the command line type mvn spring-boot:run. This will start the server. You should see this output indicating the server is running:  

```
Tomcat started on port(s): 8081 (http)
```  
## 3. Discovery

现在我们有了配置服务器，我们需要一个方法，使得所有的服务可以互相发现。解决这个问题通过设置Eureka Discovery Server 发现服务。既然应用可以运行在任何主机端口，我们需要一个注册中心，作为一个地址查询应用。   

当新增加一个服务的时候，这个服务会跟发现服务交流，并且注册自己的地址，来让其他的服务发现自己，和自己交流通信。这个方法，其他的应用可以消费这个信息当他们请求的时候。   

想要学习更多关于发现服务的知识，看[Spring Cloud Eureka article](http://www.baeldung.com/spring-cloud-netflix-eureka)    

### 3.1. Setup  

start.spring.io 自动方法

Again we’ll navigate to start.spring.io. Set the artifact to ‘discovery’. Search for “eureka server” and add that dependency. Search for “config client” and add that dependency. Generate the project.

手动Maven Pom方法

Alternatively, we can create a Spring Boot project, copy the contents of the POM from config server and swap in these dependencies:  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

For reference: you will find the bundles on Maven Central ([config-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-server](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka-server%22)).

### 3.2. Spring Config  

```
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryApplication {...}
```

@EnableEurekaServer 注解将会配置这个服务作为使用Netflix Eureka的发现服务。Spring Boot 将会自动的探测配置依赖在classpath中，并且从配置服务器中查找配置。   


### 3.3. Properties   


Now we will add two properties files:

bootstrap.properties in src/main/resources  

```
spring.cloud.config.name=discovery
spring.cloud.config.uri=http://localhost:8081
```  

These properties will let discovery server query the config server at startup.

bootstarp.properties 中的配置会在应用刚刚启动的时候加载，这时应用还没有启动完成，应用会从config server （配置仓库）中获取discovery.properties的配置内容。

discovery.properties in our Git repository  

discovery.properties的内容如下。  

```
spring.application.name=discovery
server.port=8082
 
eureka.instance.hostname=localhost
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```  

The filename must match the spring.application.name property.  

这个文件名必须和spring.application.name的属性匹配。  

还要说一下，我们告诉这个服务，在deafult zone运行，这个属性会和config client的region setting匹配。我们还告诉这个server,不要向其他的发现服务实例注册。  

在生产环境，你会有不止一个的发现服务来提供冗余在服务器故障事故中，这个时候这里的配置就要换成true，不能还是FALSE。   

还有config server只能发现提交的配置文件，所以配置git仓库必须要提交。  

### 3.4. Add Dependency to the Config Server  给config server 添加依赖  

Add this dependency to the config server POM file:  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```  

For reference: you will find the bundle on Maven Central ([eureka-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22)).  

Add these properties to the application.properties file in src/main/resources of the config server:  

把config server 配置成Eureka客户端，application.properties再加点配置。

```
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

### 3.5. Run  

Start the discovery server using the same command, mvn spring-boot:run. The output from the command line should include:  

启动发现服务 

```
Fetching config from server at: http://localhost:8081
...
Tomcat started on port(s): 8082 (http)
```  

Stop and rerun the config service. If all is good output should look like:

重启config service

```
DiscoveryClient_CONFIG/10.1.10.235:config:8081: registering service...
Tomcat started on port(s): 8081 (http)
DiscoveryClient_CONFIG/10.1.10.235:config:8081 - registration status: 204
```

## 4. Gateway 网关  

到目前为止，我们解决了配置和发现的问题，我们仍然有一个客户端访问所有应用的问题。  

如果我们把所有的东西都放在分布式系统中，我们将面临管理复杂的CORS headers来允许所有客户端的跨域访问问题（cross-origin requests）。我们解决这个问题，通过一个Gateway Server. 网关服务将会作为一个反向代理，在客户端（如，浏览器）和我们的后台服务之间来回数据通信。  

一个网关服务，在微服务架构中是一个极好的应用。因为网关服务允许了用一个单一的host来响应所有的客户端请求。这会消除对CORS的需求，还有给了我们一个非常方便的地方来处理各个服务相同的问题，比如认证authentication。  

### 4.1. Setup

By now we know the drill. Navigate to start.spring.io. Set the artifact to ‘gateway‘. Search for “zuul” and add that dependency. Search for “config client” and add that dependency. Search for “eureka discovery” and add that dependency. Generate that project.

Alternatively, we could create a Spring Boot app with these dependencies:  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```  

For reference: you will find the bundle on Maven Central ([config-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22), [zuul](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-zuul%22)).  

### 4.2. Spring Config  

Now we will add two properties files:

bootstrap.properties in src/main/resources  

```
spring.cloud.config.name=gateway
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/

```  
gateway.properties in our Git repository  

```
spring.application.name=gateway
server.port=8080
 
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
 
zuul.routes.book-service.path=/book-service/**
zuul.routes.book-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.book-service.execution.isolation.thread.timeoutInMilliseconds=600000
 
zuul.routes.rating-service.path=/rating-service/**
zuul.routes.rating-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.rating-service.execution.isolation.thread.timeoutInMilliseconds=600000
 
zuul.routes.discovery.path=/discovery/**
zuul.routes.discovery.sensitive-headers=Set-Cookie,Authorization
zuul.routes.discovery.url=http://localhost:8082
hystrix.command.discovery.execution.isolation.thread.timeoutInMilliseconds=600000

```  

zuul.routes 属性允许我们定义一个应用来负责一些特定的基于url的请求，我们的属性告诉Zuul,把所有的符合‘/book-service/**’的请求映射到book-service这个应用。Zuul之后将会查找book-service的host,通过发现服务器，然后把请求转发到获取的host上。  

记住git仓库的配置修改后要提交。  

### 4.4. Run 网关服务器运行  

运行config server和discover server,然后等待config server注册到了discover server.如果他们已经在运行，不必重启他们。一旦这边完成，启动网关服务。网关服务应该在8080端口启动，并且把自己注册到发现服务上。终端的输出中应该包含以下：   

```
Fetching config from server at: http://10.1.10.235:8081/
...
DiscoveryClient_GATEWAY/10.1.10.235:gateway:8080: registering service...
DiscoveryClient_GATEWAY/10.1.10.235:gateway:8080 - registration status: 204
Tomcat started on port(s): 8080 (http)
```  

一个很容易犯的错误是，在配置服务启动完成之前启动网关服务。将会看到如下日志：  

```
Fetching config from server at: http://localhost:8888
```  

这是配置服务默认的url和端口，这表明当网关服务请求获取配置的时候，发现服务并没有配置服务的地址。等几秒钟，再试一次，一旦配置服务注册在Eureka上，这个问题就没了。   


## 5. Book Service 

在微服务架构中，我们随意的做出许多应用来满足业务目标。通常情况下，工程师通过域名分割他们的服务。我们将会按照这个模式来创建自己的图书服务，来处理所有的关于图书的操作。  

### 5.1. Setup

One more time. Navigate to start.spring.io. Set the artifact to ‘book-service’. Search for “web” and add that dependency. Search for “config client” and add that dependency. Search for “eureka discovery” and add that dependency. Generate that project.  

Alternatively, add these dependencies to a project:  

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```  

For reference: you will find the bundle on Maven Central ([config-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-client](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22), [web](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)).  

### 5.2. Spring Config  

Let’s modify our main class:  

```
@SpringBootApplication
@EnableEurekaClient
@RestController
@RequestMapping("/books")
public class BookServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookServiceApplication.class, args);
    }
 
    private List<Book> bookList = Arrays.asList(
        new Book(1L, "Baeldung goes to the market", "Tim Schimandle"),
        new Book(2L, "Baeldung goes to the park", "Slavisa")
    );
 
    @GetMapping("")
    public List<Book> findAllBooks() {
        return bookList;
    }
 
    @GetMapping("/{bookId}")
    public Book findBook(@PathVariable Long bookId) {
        return bookList.stream().filter(b -> b.getId().equals(bookId)).findFirst().orElse(null);
    }
}
```
We also added a REST controller and a field set by our properties file to return a value we will set during configuration.

Let’s now add the book POJO:

```
public class Book {
    private Long id;
    private String author;
    private String title;
 
    // standard getters and setters
}
```  

### 5.3. Properties    

Now we just need to add our two properties files:

bootstrap.properties in src/main/resources:
  

```
spring.cloud.config.name=book-service
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```  

book-service.properties in our Git repository:  

```
spring.application.name=book-service
server.port=8083
 
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```  

Let’s commit the changes to the repository.

### 5.4. Run 自定义的业务服务运行  

Once all the other applications have started we can start the book service. The console output should look like:  

```
DiscoveryClient_BOOK-SERVICE/10.1.10.235:book-service:8083: registering service...
DiscoveryClient_BOOK-SERVICE/10.1.10.235:book-service:8083 - registration status: 204
Tomcat started on port(s): 8083 (http)
```  

Once it is up we can use our browser to access the endpoint we just created. Navigate to http://localhost:8080/book-service/books and we get back a JSON object with two books we added in out controller. Notice that we are not accessing book service directly on port 8083 but we are going through the gateway server.  


## 6. Rating Service 评分服务

不多说，跟图书服务类似

## 7. Conclusion 总结  

现在我们能够连接spring cloud的多个部分到一个运行的微服务应用。这篇文章形成了一个基础，我们可以用来构建更复杂的应用。  

As always, you can find this source code over on [Github](https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-bootstrap). 



```

Tips & Tricks

[download-a-single-folder-or-directory-from-a-github-repo](http://stackoverflow.com/questions/7106012/download-a-single-folder-or-directory-from-a-github-repo)




 

