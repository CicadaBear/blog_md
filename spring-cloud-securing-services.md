# Spring Cloud – Securing Services

From [spring-cloud-securing-services](http://www.baeldung.com/spring-cloud-securing-services)  

## 1. Overview 概览 

在之前的文章中，[ Spring Cloud – Bootstrapping](http://www.baeldung.com/spring-cloud-bootstrapping),我们已经创建了一个基本的Spring Cloud 应用。这篇文章展示的是怎样为它加上安全控制。  

我们将自然的采用Spring Security来通过Spring Session和Redis共享session.这个方法容易设置并且容易拓展到许多业务情景。如果还不熟悉 Spring Session,check out [this article](http://www.baeldung.com/spring-session).

共享session让我们有能力了在gateway service上登录用户并把认证传递到系统上的任何其他服务。  

如果对Redis或者Spring Security不熟悉。好方法是对这些话题做一个快速的回顾。虽然文章中很多是复制粘贴的，但是理解下边是怎么发生的，还是不可替代的。  

Redis教程[spring-data-redis-tutorial](http://www.baeldung.com/spring-data-redis-tutorial) . Spring Security教程[spring-security-login](http://www.baeldung.com/spring-security-login), [role-and-privilege-for-spring-security-registration](http://www.baeldung.com/role-and-privilege-for-spring-security-registration), and [spring-security-session](http://www.baeldung.com/spring-security-session). To get a complete understanding of Spring Security, have a look at the learn-spring-security-the-master-class.   

## 2. Maven Setup 设置Maven  

第一步，为每一个module加上spring-boot-starter-security依赖:   

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```  
因为我们使用Spring依赖管理，我们可以忽略spring-boot-starter版本.  

第二步，为每个module添加 spring-session, spring-boot-starter-data-redis 依赖：  

```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```   
只有4个应用将会跟Spring Session绑定， discovery, gateway, book-service, and rating-service.下一步，在这4个应用相同的文件夹下添加一个session 配置类作为主应用文件：  

```
@EnableRedisHttpSession
public class SessionConfig
  extends AbstractHttpSessionApplicationInitializer {
}
```

最后，在这4个应用的git配置仓库properties文件中添加一下内容：  

```
spring.redis.host=localhost 
spring.redis.port=6379
```
现在准备配置每个应用各自特殊的部分。  

## 3. Securing Config Service Config Service安全配置  

这个配置服务包含敏感的信息，跟数据库连接相关的，API keys.这些信息不可妥协，开始深入安全配置。   

添加如下安全配置内容到config service resources文件夹下的application.properties文件:  

```
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
security.user.name=configUser
security.user.password=configPassword
security.user.role=SYSTEM
```

This will set up our service to login with discovery. In addition, we are configuring our security with the application.properties file.

这些配置将会设置config service用服务发现登录。  

下边配置discovery service.  

## 4. Securing Discovery Service Discovery Service安全配置  

Discovery Service保存着敏感的信息，关于应用中所有服务的地址。它还可以注册这些服务的新实例。  

如果恶意的客户端获取访问，他们讲获取系统中所有服务的地址并且能够注册他们自己的恶意服务到我们的应用中。为Discovery Service加上安全配置非常严峻。  

### 4.1. Security Configuration 安全配置  

添加一个security filter 安全过滤器来保护相关的“url”,其他服务中用到的：  

```
@Configuration
@EnableWebSecurity
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 
   @Autowired
   public void configureGlobal(AuthenticationManagerBuilder auth) {
       auth.inMemoryAuthentication().withUser("discUser")
         .password("discPassword").roles("SYSTEM");
   }
 
   @Override
   protected void configure(HttpSecurity http) {
       http.sessionManagement()
         .sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
         .and().requestMatchers().antMatchers("/eureka/**")
         .and().authorizeRequests().antMatchers("/eureka/**")
         .hasRole("SYSTEM").anyRequest().denyAll().and()
         .httpBasic().and().csrf().disable();
   }
}
```
这些配置会给discovery service设置一个SYSTEM 用户。This is a basic Spring Security configuration with a few twists. Let’s take a look at those twists: 这是一个稍作调整的基本Spring Security配置，看一下做的调整：  

* @Order(1) - 告诉Spring首先过这个过滤器，这个过滤器会在其他过滤器前边执行  
* .sessionCreationPolicy - 告诉Spring总是在这个filter上创建session当用户登录的时候。  
* .requestMatchers - 限定这个过滤器过滤哪些url.  

这个security filter,配置一个孤立的认证环境，只适合discovery service。  

### 4.2. Securing Eureka Dashboard Eureka仪表盘安全配置  

既然我们的discovery application 有了一个很棒的UI来查看当前注册的服务，我们暴露这个url,通过再用一个security filter并且把它和其他的应用的认证绑定。 记住，没有@Order()注解时，这个security filter就是最后一个security filter被评估（是不是match url匹配url）。  

```
@Configuration
public static class AdminSecurityConfig
  extends WebSecurityConfigurerAdapter {
 
@Override
protected void configure(HttpSecurity http) {
   http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.NEVER)
     .and().httpBasic().disable().authorizeRequests()
     .antMatchers(HttpMethod.GET, "/").hasRole("ADMIN")
     .antMatchers("/info", "/health").authenticated().anyRequest()
     .denyAll().and().csrf().disable();
   }
}
```

把这个配置静态类添加到SecurityConfig类中。这将会创建第二个security filter,这个过滤器将会控制Eureka UI的访问。 这个过滤器有几个特性，下边看一下这几个特性：  

* httpBasic().disable() - 告诉 spring security为这个filter禁用所有的认证程序。也就是，这filter不具备认证功能，也就是，必须在其他有认证功能的应用（过滤器）认证完成，这里的UI才有可能可以访问。  
* sessionCreationPolicy - session创建策略，我们设置成了NEVER,来表明我们要求用户必须在之前认证成功，才能访问被这个过滤器保护的资源。  

这个filter将不会设置user session并且依赖于Redis传递一个共享的security context. 同样的，它依赖于另一个服务，网关服务 gateway,来提供认证信息。  

### 4.3. Authenticating with Config Service 

在发现服务应用中，添加两个properties到bootstrap.properties:  

```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
```
这两个属性将会让发现服务认证连接到config service在启动的时候。  

更新一下git配置仓库中的discovery.properties. 

```
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```
We have added basic authentication credentials to our discovery service to allow it to communicate with the config service. 我们已经添加了basic认证到发现服务，来允许它和config server进行通信（我不知道说的是哪，这句话怎么都别扭）。另外，我们还配置Eureka运行在单机模式，通过告诉服务不要向自己登记。  

提交git repositroy来让配置生效。  

## 5. Securing Gateway Service  安全控制网关服务  

网关服务是唯一我们想要向世界暴露的服务。 同样的，它将需要安全来确保只有认证的用户可以访问敏感的信息。  

### 5.1. Security Configuration 安全配置  

向在发现服务中一样创建一个SecurityConfig类，并且overwrite这几个方法，像这样。   

```
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) {
    auth.inMemoryAuthentication().withUser("user").password("password")
      .roles("USER").and().withUser("admin").password("admin")
      .roles("ADMIN");
}
 
@Override
protected void configure(HttpSecurity http) {
    http.authorizeRequests().antMatchers("/book-service/books")
      .permitAll().antMatchers("/eureka/**").hasRole("ADMIN")
      .anyRequest().authenticated().and().formLogin().and()
      .logout().permitAll().logoutSuccessUrl("/book-service/books")
      .permitAll().and().csrf().disable();
}
```   
这个配置非常直接。我们声明了一个security filter,并带有login form，这个过滤器保护着很多endpoints.  

这个security在 /eureka/** 是保护一些静态资源，我们将会提供这些静态资源在网关服务上，这些静态资源是为Erueka status page准备的。如果你是按着文章来构建项目，复制 resource/static 文件夹从网关项目的的github到自己的项目。  

现在，来修改config类上的@EnableRedisHttpSession注解。  

```
@EnableRedisHttpSession(
  redisFlushMode = RedisFlushMode.IMMEDIATE)
```  
把flush mode设置为了immediate，是为了立刻持久化session上任何改变。这在为跳转准备认证token上有帮助。  

最后，加上一个ZuulFilter,这个filter会在登录之后向后传递authentication token.  

```
@Component
public class SessionSavingZuulPreFilter
  extends ZuulFilter {
 
    @Autowired
    private SessionRepository repository;
 
    @Override
    public boolean shouldFilter() {
        return true;
    }
 
    @Override
    public Object run() {
        RequestContext context = RequestContext.getCurrentContext();
        HttpSession httpSession = context.getRequest().getSession();
        Session session = repository.getSession(httpSession.getId());
 
        context.addZuulRequestHeader(
          "Cookie", "SESSION=" + httpSession.getId());
        return null;
    }
 
    @Override
    public String filterType() {
        return "pre";
    }
 
    @Override
    public int filterOrder() {
        return 0;
    }
}
```   

这filter将会获取request，当登录后request被重定向时，并且添加上session key作为header中的cookie.这将会在登录之后，传播认证信息到任何的后台应用上。  

### 5.2. Authenticating with Config and Discovery Service  

加上以下配置信息到网关服务的bootstrap.properties文件。  

```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
```  

更新一下，git配置仓库中的gateway.properties。  

```
management.security.sessions=always
 
zuul.routes.book-service.path=/book-service/**
zuul.routes.book-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.book-service.execution.isolation.thread
    .timeoutInMilliseconds=600000
 
zuul.routes.rating-service.path=/rating-service/**
zuul.routes.rating-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.rating-service.execution.isolation.thread
    .timeoutInMilliseconds=600000
 
zuul.routes.discovery.path=/discovery/**
zuul.routes.discovery.sensitive-headers=Set-Cookie,Authorization
zuul.routes.discovery.url=http://localhost:8082
hystrix.command.discovery.execution.isolation.thread
    .timeoutInMilliseconds=600000
```  

我们加上了session management为always 生成session,因为我们只有一个security filter,可以在properties文件中设置。下一步，我们添加redis host和server properties.  

另外，我们加上了一个route,这个路由将会重定向request到discovery service.既然单机模式的发现服务将不会注册自己，我们必须定位发现服务通过url.  

我们可以移除掉serviceUrl.defaultZone property从git 配置仓库的gateway.properties文件中。这个值和bootstrap的重复了。  

提交git仓库，让修改生效。  

## 6. Securing Book Service 安全控制Book Service  

book service将包含被很多用户控制的敏感信息。这个service必须被保护来防止敏感信息泄露。  

### 6.1. Security Configuration 安全配置  

为安全控制book service我们将复制SecurityConfig类从网关服务，并且重新如下方法:  

```
@Override
protected void configure(HttpSecurity http) {
    http.httpBasic().disable().authorizeRequests()
      .antMatchers("/books").permitAll()
      .antMatchers("/books/*").hasAnyRole("USER", "ADMIN")
      .authenticated().and().csrf().disable();
}
```  

### 6.2. Properties 配置  

添加这些配置到 book service的bootstrap.porperties文件。  

```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
```  

Let’s add properties to our book-service.properties file in our git repository:  

```
management.security.sessions=never
```  

删除 book-service.properties中的serviceUrl.defaultZone。  

We can remove the serviceUrl.defaultZone property from the book-service.properties file in our configuration git repository. This value is duplicated in the bootstrap file.

Remember to commit these changes so the book-service will pick them up.  

## 7. Securing Rating Service

The rating service also needs to be secured.  


### 7.1. Security Configuration

To secure our rating service we will copy the SecurityConfig class from the gateway and overwrite the method with this content:

复制网关服务的安全配置类，修改如下：  
  

```
@Override
protected void configure(HttpSecurity http) {
    http.httpBasic().disable().authorizeRequests()
      .antMatchers("/ratings").hasRole("USER")
      .antMatchers("/ratings/all").hasAnyRole("USER", "ADMIN").anyRequest()
      .authenticated().and().csrf().disable();
}  
```

We can delete the configureGlobal() method from the gateway service. 我们可以将网关中的configureGlobal()删除。（我都不知道为什么冒出这么一句，理论上，图书服务，评分服务，自己不具备认证功能，事实上，他们的例子中也是这么做的，把网关中的configureGlobal()删了，谁负责提供用户Authentication信息，真不知道为什么冒出这么一句）。  

### 7.2. Properties  

Add these properties to the bootstrap.properties file in src/main/resources of the rating service:  

```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
```  

Let’s add properties to our rating-service.properties file in our git repository:  

```
management.security.sessions=never
```  

We can remove the serviceUrl.defaultZone property from the rating-service.properties file in our configuration git repository. This value is duplicated in the bootstrap file.   

Remember to commit these changes so the rating service will pick them up.  

### 8. Running and Testing 运行测试  

开启Redis和所有的services,config, discovery, gateway, book-service, and rating-service,测试一下。  

First, let’s create a test class in our gateway project and create a method for our test:  

```
public class GatewayApplicationLiveTest {
    @Test
    public void testAccess() {
        ...
    }
}
```  
Next, let’s set up our test and validate that we can access our unprotected /book-service/books resource by adding this code snippet inside our test method:  

测试可以访问不受保护的资源,在test方法中加几行代码:  

```
TestRestTemplate testRestTemplate = new TestRestTemplate();
String testUrl = "http://localhost:8080";
 
ResponseEntity<String> response = testRestTemplate
  .getForEntity(testUrl + "/book-service/books", String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```  

运行test,验证结果。如果看到失败，确认一下整个应用启动成功了，并且配置从配置服务中加载了。  

现在测试用户将会被重定向到login,当没有认证的用户访问受保护的资源。继续在test方法最后添加代码：  

```
response = testRestTemplate
  .getForEntity(testUrl + "/book-service/books/1", String.class);
Assert.assertEquals(HttpStatus.FOUND, response.getStatusCode());
Assert.assertEquals("http://localhost:8080/login", response.getHeaders()
  .get("Location").get(0));
```  
再次运行test,确认成功。  

Next, let’s actually log in and then use our session to access the user protected result:  

登录用户：  

```
MultiValueMap<String, String> form = new LinkedMultiValueMap<>();
form.add("username", "user");
form.add("password", "password");
response = testRestTemplate
  .postForEntity(testUrl + "/login", form, String.class);
```

now, let us extract the session from the cookie and propagate it to the following request:  

提取登录session cookie：  

```
String sessionCookie = response.getHeaders().get("Set-Cookie")
  .get(0).split(";")[0];
HttpHeaders headers = new HttpHeaders();
headers.add("Cookie", sessionCookie);
HttpEntity<String> httpEntity = new HttpEntity<>(headers);
```  

and request the protected resource:  

带着认证cookie,请求：  

```
response = testRestTemplate.exchange(testUrl + "/book-service/books/1",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```  

Run the test again to confirm the results.  

Now, let’s try to access the admin section with the same session:  

```
response = testRestTemplate.exchange(testUrl + "/rating-service/ratings/all",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.FORBIDDEN, response.getStatusCode());
```  

再次test,确认成功。  

Now, let’s try to access the admin section with the same session:  

用普通用户，访问admin的资源：  

```
response = testRestTemplate.exchange(testUrl + "/rating-service/ratings/all",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.FORBIDDEN, response.getStatusCode());
```

Run the test again, and as expected we are restricted from accessing admin areas as a plain old user.  

预期的是重定向到登录页面。  

The next test will validate that we can log in as the admin and access the admin protected resource: 

测试admin用户访问admin资源：


```
form.clear();
form.add("username", "admin");
form.add("password", "admin");
response = testRestTemplate
  .postForEntity(testUrl + "/login", form, String.class);
 
sessionCookie = response.getHeaders().get("Set-Cookie").get(0).split(";")[0];
headers = new HttpHeaders();
headers.add("Cookie", sessionCookie);
httpEntity = new HttpEntity<>(headers);
 
response = testRestTemplate.exchange(testUrl + "/rating-service/ratings/all",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```  

Our test is getting big! But we can see when we run it that by logging in as the admin we gain access to the admin resource.

Our final test is accessing our discovery server through our gateway. To do this add this code to the end of our test:  

最后，测试一下，发现服务的状态页面：  

```
response = testRestTemplate.exchange(testUrl + "/discovery",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
```
Run this test one last time to confirm that everything is working. Success!!!  

Did you miss that? Because we logged in on our gateway service and viewed content on our book, rating, and discovery services without having to log in on four separate servers!  

你错过了吗？（？？？），因为我们在网关登录，然后访问了所有分离的服务上受保护的资源。  

By utilizing Spring Session to propagate our authentication object between servers we are able to log in once on the gateway and use that authentication to access controllers on any number of backing services.   

通过，工具化Spring Session来传播认证object,在服务之间，我们可以登录一次在网关，然后用认证信息访问所有的后台服务。  

## 9. Conclusion总结  

Security in the cloud certainly becomes more complicated. But with the help of Spring Security and Spring Session, we can easily solve this critical issue.  


We now have a cloud application with security around our services. Using Zuul and Spring Session we can log users in only one service and propagate that authentication to our entire application. This means we can easily break our application into proper domains and secure each of them as we see fit.  


安全在云应用中肯定变得更复杂了，但是在Spring Security和Spring Session的帮助下，我们可以很容易解决关键问题。  

我们现在有了一个云应用，服务受到安全的保护。使用Zuul和Spring Session我们可以在一个服务上登录用户，然后把认证信息传播到整个应用上。这意味着，我们可以简单的把应用部署到合适的域，并给他们加上合适的安全控制。

As always you can find the source code on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-bootstrap).

***