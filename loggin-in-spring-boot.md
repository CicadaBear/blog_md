
# **Logging in Spring Boot**

[From-baeldung](https://www.baeldung.com/logback)

## **1. Overview  概览**

In this short tutorial, we’re going to explore the main logging options available in Spring Boot.   

在这个简短的教程中，我们将要探索Sping Boot中主要的日志可选方案。

Deeper information about Logback is available in [A Guide To Logback](https://www.baeldung.com/logback), while Log4j2 is introduced in [Intro to Log4j2 – Appenders, Layouts and Filters](https://www.baeldung.com/log4j2-appenders-layouts-filters).  

关于Logback更深的信息在[A Guide To Logback](https://www.baeldung.com/logback)，介绍Log4j2的文章在[Intro to Log4j2 – Appenders, Layouts and Filters](https://www.baeldung.com/log4j2-appenders-layouts-filters)。

## **2. Initial Setup 初始化设置**  

Let’s first create a Spring Boot module — the recommended way to do so is using [Spring Initializr](https://start.spring.io/), which we cover in our [Spring Boot Tutorial](https://www.baeldung.com/spring-boot-start).  

首先创建一个Spring Boot module - 推荐的方法是使用Sping Initializr, 我们在[Spring Boot Tutorial](https://www.baeldung.com/spring-boot-start)的教程中讲到了。

Now let’s create our only class file, LoggingController: 

现在创建我们唯一的class文件，LoggingController:   

```Java  
@RestController
public class LoggingController {
 
    Logger logger = LoggerFactory.getLogger(LoggingController.class);
 
    @RequestMapping("/")
    public String index() {
        logger.trace("A TRACE Message");
        logger.debug("A DEBUG Message");
        logger.info("An INFO Message");
        logger.warn("A WARN Message");
        logger.error("An ERROR Message");
 
        return "Howdy! Check out the Logs to see the output...";
    }
}  
```  
Once we’ve loaded the web application, we’ll be able to trigger those logging lines by simply visiting http://localhost:8080/ 

一旦我们加载了web应用，我们可以通过访问 http://localhost:8080/ 这个链接来触发logging.  

## **3. Zero Configuration Logging**  

Spring Boot is a very helpful framework — it helps us forget about the majority of the configuration settings, most of which it opinionatedly autotunes.  

Spring Boot是个非常有帮助的框架，它帮我们忘记大多数的配置设置，大多数情况下它是固执己见的自动调优。 

In the case of logging, the only mandatory dependency is Apache Commons Logging.  

在这种logging例子中，仅有的强制依赖是 Apache Commons Logging 。

We need to import it only when using Spring 4.x (Spring Boot 1.x), since in Spring 5 (Spring Boot 2.x) it’s provided by Spring Framework’s spring-jcl module.  

我们只需要导入它，当我们使用 Spring 4.x (Spring Boot 1.x)， 既然 Spring 5 (Spring Boot 2.x)，Spring 框架提供了 **spring-jcl** 模块。  

**We shouldn’t worry about importing spring-jcl at all if we’re using a Spring Boot Starter (which almost always we are)**. That’s because every starter, like our spring-boot-starter-web, depends on spring-boot-starter-logging, which already pulls in spring-jcl for us.  

我们一点也不应该担心导入 spring-jcl，如果我们使用 Spring Boot Starter (但是我们几乎总是担心)。 那是因为每一个starter, 比如 spring-boot-starter-web, 依赖于 spring-boot-starter-logging，已经为我们导入了。 

When using starters, Logback is used for logging by default.  

当我们使用starters时，Logback 默认用来logging。 

Spring Boot pre-configures it with patterns and ANSI colors to make the standard output more readable.  

Spring Boot 预配置了logging, with 模式和ANSI颜色使标准输出更具可读性。  

Let’s now run the application and visit the http://localhost:8080/ page, and see what happens in the console:  

让我们现在运行一下应用然后访问一下 http://localhost:8080/ 这个页面，看看终端发生了什么。  

![](https://cicadabearblog.oss-cn-shenzhen.aliyuncs.com/logback-default-logging-1024x562.png)  

As we can see in the above picture, **the default logging level of the Logger is preset to INFO, meaning that TRACE and DEBUG messages are not visible.**  

正如我们所看到的，默认的logging级别已经预设为了INFO,意味着TRACE和DEBUG信息是看不到的。

In order to activate them without changing the configuration, we can pass the –debug or –trace arguments on the command line:  

为了激活他们在不改配置的情况下，我们可以传入 -debug或者 -trace 参数在命令行： 

    java -jar target/spring-boot-logging-0.0.1-SNAPSHOT.jar --trace  

If we want to change the verbosity permanently, we can do so in the application.properties file as described here:  

如果我们想永久的改变啰嗦级别，我们可以在application.properties配置文件中改，如下：  


    logging.level.root=WARN
    logging.level.com.baeldung=TRACE

## **4. Logback Configuration Logging**  

Even though the default configuration is useful (for example to get started in zero time during POCs or quick experiments), it’s most likely not enough for our daily needs.  

尽管默认的配置很有用（例如， get started in zero time during POCs或者 快速实验），但是它很可能不能满足我们日常需求。  

Let’s see **how to include a Logback configuration** with a different color and logging pattern, with separate specifications for console and file output, and with a decent rolling policy to avoid generating huge log files.  

让我看看怎样引入Logback配置，with 不同的颜色，不同的logging模式，with 分隔开的技术参数对于控制台和文件输出，并且 with 一个合理的rolling政策，来避免产生巨大的日志文件。 

First of all, we should go toward a solution which allows handling our logging settings alone, instead of polluting application.properties, which is commonly used for many other application settings.  

首先，我们应该朝着一个解决方案的方向，什么解决方案，允许我们单独处理logging设置，而不是污染application.properties，application.properties文件常被用于许多的其他应用配置。 

When a file in the classpath has one of the following names, Spring Boot will automatically load it over the default configuration:  

当一个文件在classpath中有如下几个名字的时候，Spring Boot将会自动将它加载为默认配置： 

* logback-spring.xml
* logback.xml 
* logback-spring.groovy
* logback.groovy  

**Spring recommends using the -spring variant** over the plain ones whenever possible, as described [here](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html#boot-features-custom-log-configuration).  

Spring推荐使用 -spring 变体，相对于没有加 -spring的，官方描述在[这](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html#boot-features-custom-log-configuration)。

Let’s write a simple logback-spring.xml:  

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
 
    <property name="LOGS" value="./logs" />
 
    <appender name="Console"
        class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %black(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender>
 
    <appender name="RollingFile"
        class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOGS}/spring-boot-logger.log</file>
        <encoder
            class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder>
 
        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- rollover daily and when the file reaches 10 MegaBytes -->
            <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
     
    <!-- LOG everything at INFO level -->
    <root level="info">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>
 
    <!-- LOG "com.baeldung*" at TRACE level -->
    <logger name="com.baeldung" level="trace" additivity="false">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </logger>
 
</configuration>

```  
And when we run the application, here’s the output: 

当我们运行的时候，输出如下： 

![](https://cicadabearblog.oss-cn-shenzhen.aliyuncs.com/logback-custom-logging-1024x576.png)  

As we can see, it now logs TRACE and DEBUG messages, and the overall console pattern is both textually and chromatically different than before.  

正如我们所见，现在输出了TRACE和DEBUG信息，并且所有的控制台模式都是文字的和上色的跟之前的不同。  

It also now logs on a file in a /logs folder created under the current path and archives it through a rolling policy.  

它也在文件夹/logs 中输出了，这个文件夹在当前目录下被创建，并且通过rolling policy实现。  


## **5. Log4j2 Configuration Logging**  

While Apache Commons Logging is at the core, and Logback is the reference implementation provided, all the routings to the other logging libraries are already included to make it easy switching to them.

当 Apache Commons Logging 在核心的时候，Logback是被提供的引用的实现，所有的到其他logging库的路径已经被引入，为了让它容易切换底层库。  

**In order to use any logging library other than Logback, though, we need to exclude it from our dependencies.**  

为了使用任意其他的loggin库，不过我们需要把它从我们的依赖中exclude掉。 

For every starter like this one (it’s the only one in our example, but we could have many of them):  

对于每一个starter像这个（在我们的例子中仅有一个starter，但是我们可以有很多个）： 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```  
we need to turn it into a skinny version, and (only once) add our alternative library, here through a starter itself: 

我们需要把starter变成一个更小的版本，并且（只有这样） 添加我们可替代的logging库，如下所示：  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```  
At this point, we need to place in the classpath a file named like one of the following:  

这个时候，我们需要放一个文件在classpath中，命名如下： 

* log4j2-spring.xml
* log4j2.xml 

We’ll print through Log4j2 (over SLF4J) without further modifications. 

我们将会通过Log4j2（在SLF4J之上）不需要更多的修改。 

Let’s write a simple log4j2-spring.xml:  

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout
                pattern="%style{%d{ISO8601}}{black} %highlight{%-5level }[%style{%t}{bright,blue}] %style{%C{1.}}{bright,yellow}: %msg%n%throwable" />
        </Console>
 
        <RollingFile name="RollingFile"
            fileName="./logs/spring-boot-logger-log4j2.log"
            filePattern="./logs/$${date:yyyy-MM}/spring-boot-logger-log4j2-%d{-dd-MMMM-yyyy}-%i.log.gz">
            <PatternLayout>
                <pattern>%d %p %C{1.} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!-- rollover on startup, daily and when the file reaches 
                    10 MegaBytes -->
                <OnStartupTriggeringPolicy />
                <SizeBasedTriggeringPolicy
                    size="10 MB" />
                <TimeBasedTriggeringPolicy />
            </Policies>
        </RollingFile>
    </Appenders>
 
    <Loggers>
        <!-- LOG everything at INFO level -->
        <Root level="info">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFile" />
        </Root>
 
        <!-- LOG "com.baeldung*" at TRACE level -->
        <Logger name="com.baeldung" level="trace"></Logger>
    </Loggers>
 
</Configuration>
```  
And when we run the application, here’s the output: 

当我们运行应用的时候，以下是输出：  

![](https://cicadabearblog.oss-cn-shenzhen.aliyuncs.com/log4j2-custom-logging-1024x562.png)  

As we can see, the output is quite different from the Logback one — a proof that we’re fully using Log4j2 now.  

正如我们所见，输出跟之前的相比已经非常不同了，这是个我们使用Log4j2的证据。 

In addition to the XML configuration, Log4j2 allows us to use also a YAML or JSON configuration, as described [here](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging-yaml-or-json-config). 

除了XML配置之外，Log4j2也允许我们使用YAML或者是JSON配置，详情请见[这里](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging-yaml-or-json-config) 。  

## **6. Log4j2 Without SLF4J**  

We can also use Log4j2 natively, without passing through SLF4J.  

我们可以原生地使用Log4j2,不用通过SLF4J。  

In order to do that, we must simply use the native classes:  

为了实现这个，我们必须简单地使用原生类:  

```
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;
// [...]
Logger logger = LogManager.getLogger(LoggingController.class);
``` 
We don’t need to perform any other modification to the standard Log4j2 Spring Boot configuration.  

我们不需要执行任何其他的修改对标准的Log4j2 Spring Boot 配置。  

We can now exploit the brand new features of Log4j2 without being stuck with the old SLF4J interface, but we’re also tied to this implementation, and we’ll need to rewrite our code when deciding to switch to another logging framework.  

我们现在可以开发一个全新的Log4j2功能，不用没法摆脱SLF4J老版接口，但是我们也可以也被绑在了这个实现上，我还需要重新代码当我们决定切换另外的logging框架时。  

## **7. Beware of Java Util Logging 知晓Java Util Logging**  

Spring Boot also supports JDK logging, through the logging.properties configuration file.  

Spring Boot也支持JDKloggin，通过logging.properties配置文件。

There are cases when it’s not a good idea to use it, though. From the documentation:  

这些是些不应该用它的例子，在这个文章中：

> There are known classloading issues with Java Util Logging that cause problems when running from an ‘executable jar’. We recommend that you avoid it when running from an ‘executable jar’ if at all possible. 
> 这些是已知的使用Java Util Logging 的问题，这些事件造成问题当以可执行 jar包来运行的时候。我们建议尽可能避免用它单我们以可执行jar包运行的时候。  

It’s also a good practice, when using Spring 4, to manually exclude commons-logging in pom.xml, to avoid potential clashes between the logging libraries. Spring 5 instead handles it automatically, hence we don’t need to do anything when using Spring Boot 2.  

这也是一个很好的实践，当使用Spring 4时，手动exclude commons-logging 在 pom文件，来避logging库之间免潜在的冲突。Spring 5 取而代之的是自动处理它，因此，我们不需要做任何事，当我们使用Sping Boot 2的时候。 

## **8. JANSI on Windows**  

While Unix-based operating systems such as Linux and Mac OS X support ANSI color codes by default, on a Windows console everything will be sadly monochromatic.  

然而，Unix-based 操作系统默认支持ANSI color codes, 在Windows终端，所有的东西都会是悲伤的单色。  

**Windows can obtain ANSI colors through a library called JANSI.**   

Windows可以获得ANSI colors通过一个库，叫做JANSI。  

**We should pay attention to the possible classloading drawbacks, though.**   

但是，我们应该注意可能的类加载的缺陷。  

We must import and explicitly activate it in the configuration like follows:  

我们必须引入，并且显示地激活它在配置文件中，如下：  

[Logback](https://logback.qos.ch/manual/layouts.html#coloring):  

```
<configuration debug="true">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <withJansi>true</withJansi>
        <encoder>
            <pattern>[%thread] %highlight(%-5level) %cyan(%logger{15}) - %msg %n</pattern>
        </encoder>
    </appender>
    <!-- more stuff -->
</configuration>
```
[Log4j2](https://logging.apache.org/log4j/2.x/manual/layouts.html#enable-jansi):  

>    ANSI escape sequences are supported natively on many platforms but are not by default on Windows. To enable ANSI support add the Jansi jar to our application and set property log4j.skipJansi to false. This allows Log4j to use Jansi to add ANSI escape codes when writing to the console.  
>         
>    NOTE: Prior to Log4j 2.10, Jansi was enabled by default. The fact that Jansi requires native code means that Jansi can only be loaded by a single class loader. For web applications this means the Jansi jar has to be in the web container’s classpath. To avoid causing problems for web applications, Log4j will no longer automatically try to load Jansi without explicit configuration from Log4j 2.10 onward.  

It’s also worth knowing that:  

* the [layout](https://logging.apache.org/log4j/2.x/manual/layouts.html) documentation page contains useful Log4j2 JANSI informations in the highlight{pattern}{style} section
* while JANSI can color the output, Spring Boot’s Banner (native or customized through the banner.txt file) will stay monochromatic  

## **9. Conclusion** 

We’ve seen the main ways to interface with the major logging frameworks from within a Spring Boot project.

We’ve also explored the main advantages and pitfalls of every solution.

As always, the full source code is available over on Github.  
   
  
<br><br>



*** 






