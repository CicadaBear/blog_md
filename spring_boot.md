## Spring Boot

最近项目中用到了Spring Boot，特意来Spring官网[spring-boot]具体学习一下。  
这不是全文翻译，只是我自己的学习记录，当然也只记录我自己认为重要的，或者是目前认为重要的。大多数地方应该是按我自己的理解进行的总结，不是原文翻译，太长，而且英文句式直接转换成的中文，不利于中文记忆，首先是无法直接记忆英文。  


### Part II. Getting started  

如果你刚刚开始学习Spring Boot,或者普通的Spring，这个章节为你准备的。在这里我们回答了，what,how,why的问题。你将会发现一个文雅的介绍Spring Boot和安装指令。我们将会build第一个Spring Boot应用，还有讨论一些核心原则。  

#### 8. Introducing Spring Boot  

Spring Boot使创建单机，生产级的基于Spring的应用变得简单，“just run”直接运行就可以。我们对Spring平台和第三方库持自以为是的看法，所以你可以以最小的麻烦学习。大多数的Spring Boot应用需要非常少的配置。  

可以使用Spring Boot来创建Java应用，创建的Java应用是可以使用Java -jar或者是更传统的war来部署。Spring官方也提供了命令行工具来运行spring脚本。  

Spring Boot的目标是， 

* 提供一个彻底的更快的，并且广泛的可理解的开始（创建）体验对所有的Spring开发。  
* Be opinionated out of the box, but get out of the way quickly as requirements start to diverge from the defaults.
* Provide a range of non-functional features that are common to large classes of projects (e.g. embedded servers, security, metrics, health checks, externalized configuration). 
* Absolutely no code generation and no requirement for XML configuration.  


### 10. Installing Spring Boot  
#### 10.1 Installation instructions for the Java developer
#### 10.1.1 Maven installation
#### 10.1.2 Gradle installation  
#### 10.2 Installing the Spring Boot CLI  
#### 10.3 Upgrading from an earlier version of Spring Boot
不多说。

### 11. Developing your first Spring Boot application
#### 11.1 Creating the POM
#### 11.2 Adding classpath dependencies  
#### 11.3 Writing the code  

**src/main/java/Example.java**

``` 
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```
##### 11.3.1 The @RestController and @RequestMapping annotations

Spring MVC中最常见的，不是Spring Boot特有的，无需说

##### 11.3.2 The @EnableAutoConfiguration annotation  

**@EnableAutoConfiguration**是类级别的注解，是Spring Boot独有的。这个注解告诉Spring Boot来猜测，来Spring来决策想怎么配置应用，基于什么决策，基于现有的依赖。**spring-boot-starter-web** 依赖添加了Tomcat和Spring MVC,此时这个自动配置将会假设你正在开发一个Web应用，并且做相应的配置。  

**Starters and Auto-Configuration**

Auto-Configuration被设计用来和“Starters”一起工作的，但是两个概念没有直接的关联关系。你可以自由选择starts之外的jar依赖，Spring Boot还是会做好自动配置的工作。  

##### 11.3.3 The “main” method  

这个Main方法只是一个标准的方法，一个遵循了Java规范的应用入口。通过调用run,将main方法委派给Spring Boot的SpringApplication。SpringApplication将会引导应用，启动应用接下来是开始自动配置Tomcat.我们需要传入**Example.class**，来告诉SpringApplication哪个是Spring主要组件。这些可变参数可通过命令行传入。

#### 11.4 Running the example   

**mvn spring-boot:run**  

#### 11.5 Creating an executable jar   

### 12. What to read next  

**What to read next**很有意思，讲了点学习方法。希望刚刚过去的一节提供给你一些Spring Boot的基础，并且把你带到了写你自己的应用的路上。如果你是一个任务导向型的开发者，你可能想要跳过当前的文档[spring.io],下载一些[getting started][guides]解决“How do I do that with Spring”问题的指导教程，Spring官方还提供了Spring Boot专用的[How-to][howto]参考文档。  

The Spring Boot repository has also a bunch of samples you can run. The samples are independent of the rest of the code (that is you don’t need to build the rest to run or use the samples).  

要不，到下一节读“Using Spring Boot”,实在没有耐心，你也可以跳过去读，“Spring Boot features".

## Part III. Using Spring Boot








***

[spring-boot]:http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/   

[spring.io]:https://spring.io/  

[guides]:https://spring.io/guides/  

[howto]:http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#howto
