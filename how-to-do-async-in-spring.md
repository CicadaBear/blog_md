# **How To Do @Async in Spring** 

[From-baeldung](https://www.baeldung.com/spring-async)  

## **1. Overview 概览**  

In this article, we’ll explore the **asynchronous execution support in Spring** – and the @Async annotation.  

在这篇文章中，我们将会探索Spring的异步执行，使用@Async注解。  

Simply put – annotating a method of a bean with @Async will make it execute in a separate thread i.e. the caller will not wait for the completion of the called method.  

简单的将用@Async注解一个bean里边的方法，将会使它运行在一个分隔的线程，比如，调用者将不用等待被调用方法执行完成。 

One interesting aspect in Spring is that the event support in the framework also has support for async processing if you want to go that route.   

一个Spring中有趣的方面是框架对事件的支持也支持异步处理，如果你想要。  

## **2. Enable Async Support 开启异步支持**  

Let’s start by **enabling asynchronous processing** with **Java configuration** – by simply adding the @EnableAsync to a configuration class:  

```
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```  
The enable annotation is enough, but as you’d expect, there are also a few simple options for configuration as well:  

这个enable注解就足够了，但是正如你会期待的，这里还有一些配置的简单的选项：  

* **annotation** – by default, @EnableAsync detects Spring’s @Async annotation and the EJB 3.1 javax.ejb.Asynchronous; this option can be used to detect other, user-defined annotation types as well  
annotation - 默认的，@EnableAsync 探测Spring的@Async注解，并且 EJB 3.1 javax.ejb.Asynchronous；这个可选项也可以被用来探测用户自定义的注解类型。  

* **mode** – indicates the type of advice that should be used – JDK proxy-based or AspectJ weaving  
mode - 指明advice应该被使用的类型，- JDK proxy-based 或者 AspectJ weaving 

* **proxyTargetClass** – indicates the type of proxy that should be used – CGLIB or JDK; this attribute has effect only if the mode is set to AdviceMode.PROXY  
proxyTargetClass - 指明了应该使用的代理的类型 - CGLIB 或者 JDK；这个属性只会在mode设置为AdviceMode.PROXY的时候起作用。 

* **order** – sets the order in which AsyncAnnotationBeanPostProcessor should be applied; by default, it runs last, just so that it can take into account all existing proxies  
order - 设置AsyncAnnotationBeanPostProcessor 会被应用的顺序，默认，最后运行，只为了，它可以考虑到所有存在的代理。（不明觉厉）  

Asynchronous processing can also be enabled using XML configuration – by using the task namespace:  

异步处理也可以通过XML配置启用，通过使用task命名空间：  

> <task:executor id="myexecutor" pool-size="5"  />  
> <task:annotation-driven executor="myexecutor"/>

## **3. The @Async Annotation**

First – let’s go over the rules – @Async has two limitations:  

首先 - 让我们复习这里的规则 - @Async 有两个限制： 

* it must be applied to public methods only 
必须使用在public方法上。

* self-invocation – calling the async method from within the same class – won’t work  
自调用 - 在同一个类中调用async方法不起作用。  

The reasons are simple – the method needs to be public so that it can be proxied. And self-invocation doesn’t work because it bypasses the proxy and calls the underlying method directly.  

原因是简单的 - 这个方法需要是public, 这样它才可能被代理。并且，自调用不起作用，因为这样会绕过代理直接调用底层的方法。 

## *3.1. Methods with void Return Type 返回值为空的方法*

Following is the simple way to configure a method with void return type to run asynchronously:  

下面是简单的方法来配置一个返回值为空的方法来实现异步运行：  

```
@Async
public void asyncMethodWithVoidReturnType() {
    System.out.println("Execute method asynchronously. "
      + Thread.currentThread().getName());
}
```  

## *3.2. Methods With Return Type 带有返回值的方法*  

@Async can also be applied to a method with return type – by wrapping the actual return in the Future:  
@Async也可以被用在带有返回值的方法上，通过用Future包装真正的返回值。  

```
@Async
public Future<String> asyncMethodWithReturnType() {
    System.out.println("Execute method asynchronously - "
      + Thread.currentThread().getName());
    try {
        Thread.sleep(5000);
        return new AsyncResult<String>("hello world !!!!");
    } catch (InterruptedException e) {
        //
    }
 
    return null;
}
```  
Spring also provides an AsyncResult class which implements Future. This can be used to track the result of asynchronous method execution.  
Spring 也提供了 AsyncResult 类，这个类实现了Future. 这个可以被用来追踪异步的方法执行结果。  

Now, let’s invoke the above method and retrieve the result of the asynchronous process using the Future object.  
现在，我们调用调用上述方法，并且获取一个异步处理返回值，用Future对象。  

```
public void testAsyncAnnotationForMethodsWithReturnType()
  throws InterruptedException, ExecutionException {
    System.out.println("Invoking an asynchronous method. "
      + Thread.currentThread().getName());
    Future<String> future = asyncAnnotationExample.asyncMethodWithReturnType();
 
    while (true) {
        if (future.isDone()) {
            System.out.println("Result from asynchronous process - " + future.get());
            break;
        }
        System.out.println("Continue doing something else. ");
        Thread.sleep(1000);
    }
}
```  
## **4. The Executor 执行器** 

By default, Spring uses a SimpleAsyncTaskExecutor to actually run these methods asynchronously. The defaults can be overridden at two levels – at the application level or at the individual method level.  

默认情况下，Spring 使用一个 SimpleAsyncTaskExecutor 来实际上异步运行这些方法。这个默认可以在两层上被覆盖，在应用级别，或者在单独的方法级别。  

### **4.1. Override the Executor at the Method Level 方法级别覆盖** 

The required executor needs to be declared in a configuration class:  

需要的executor需要被声明在配置类中： 

```
@Configuration
@EnableAsync
public class SpringAsyncConfig {
     
    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```  
Then the executor name should be provided as an attribute in @Async:  

然后执行器名称应该被提供作为@Async的一个属性： 

```
@Async("threadPoolTaskExecutor")
public void asyncMethodWithConfiguredExecutor() {
    System.out.println("Execute method with configured executor - "
      + Thread.currentThread().getName());
}
```  

### **4.2. Override the Executor at the Application Level**  

The configuration class should implement the AsyncConfigurer interface – which will mean that it has the implement the getAsyncExecutor() method. It’s here that we will return the executor for the entire application – this now becomes the default executor to run methods annotated with @Async:  

这个配置类应当实现AsyncConfigurer接口，这意味着它实现了getAsyncExecutor()方法。在这里我们将会返回executor对整个的应用 - 这个executor成了默认的executor来运行@Async注解的方法。  

```
@Configuration
@EnableAsync
public class SpringAsyncConfig implements AsyncConfigurer {
     
    @Override
    public Executor getAsyncExecutor() {
        return new ThreadPoolTaskExecutor();
    }
     
}
```  

## **5. Exception Handling 异常处理**  

When a method return type is a Future, exception handling is easy – Future.get() method will throw the exception.

当一个方法返回值是一个Future, 异常处理非常简单 - Future.get() 将会抛出异常。  

But, if the return type is void, **exceptions will not be propagated to the calling thread**. Hence we need to add extra configurations to handle exceptions.  

但是如果一个返回值是空的方法，异常将不会被传播到调用方法。因此，我们需要添加额外的配置来处理这些异常。  

We’ll create a custom async exception handler by implementing AsyncUncaughtExceptionHandler interface. The handleUncaughtException() method is invoked when there are any uncaught asynchronous exceptions:   

我们将创建一个自定义的async 异常处理通过实现 AsyncUncaughtExceptionHandler 接口。这 handleUncaughtException()方法，被调用当这里有任何没有被捕获的异步异常时：  


```
public class CustomAsyncExceptionHandler
  implements AsyncUncaughtExceptionHandler {
 
    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {
  
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }
     
}
```  

In the previous section, we looked at the AsyncConfigurer interface implemented by the configuration class. As part of that, we also need to override the getAsyncUncaughtExceptionHandler() method to return our custom asynchronous exception handler:  

在之前的章节中，我们看过了 AsyncConfigurer 接口被配置类实现。作为这个的一部分，我们需要重写 getAsyncUncaughtExceptionHandler() 方法来返回我们的自定义异步异常处理：

```
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();
}
```  

## **6. Conclusion**  

In this tutorial, we looked at running asynchronous code with Spring. We started with the very basic configuration and annotation to make it work but also looked at more advanced configs such as providing our own executor, or exception handling strategies. 

[Youtube-Video-Tutorial](https://youtu.be/lorTmWJp-z4) 

And, as always, the full code presented in this article is available over on [Github](https://github.com/eugenp/tutorials/tree/master/spring-all).









<br><br><br><br><br><br><br><br>




