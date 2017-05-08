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




  




