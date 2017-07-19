# Lambda Expressions and Functional Interfaces: Tips and Best Practices

From [java-8-lambda-expressions-tips](http://www.baeldung.com/java-8-lambda-expressions-tips)  

## 1. Overview 概览

Now that Java 8 has reached wide usage, patterns, and best practices have begun to emerge for some of its headlining features. In this tutorial, we will take a closer look to functional interfaces and lambda expressions.  

现在Java 8达到了广泛的应用，模式并且最好的实践开始浮现对于一些Java 8的显眼的功能。 在这篇教程中，我们将看一下functional interfaces和lambda表达式。  

## 2. Prefer Standard Functional Interfaces 更喜欢标准的函数式接口  

Functional interfaces, which are gathered in the java.util.function package, satisfy most developers’ needs in providing target types for lambda expressions and method references. Each of these interfaces is general and abstract, making them easy to adapt to almost any lambda expression. Developers should explore this package before creating new functional interfaces.  

Functional interfaces函数接口，被收集在了java.util.function包中，满足大多数开发者在为lambda表达式和方法引用提供目标类型的需求。这些接口中的额每一个都是普通的和抽象的，使它们很容易适配几乎所有的lambda表达式。开发者应该探究这个包在创建新的函数式接口之前。

Consider an interface Foo:

```

	@FunctionalInterface
	public interface Foo {
    	String method(String string);
	}
```

and a method add() in some class UseFoo, which takes this interface as a parameter:  

在其他类里边的一个add()方法，把这个接口作为一个参数： 

```

	public String add(String string, Foo foo) {
    	return foo.method(string);
	}
```

To execute it, you would write:  

为执行它，这样写

```

	Foo foo = parameter -> parameter + " from lambda";
	String result = useFoo.add("Message ", foo);
```

Look closer and you will see that Foo is nothing more than a function that accepts one argument and produces a result. Java 8 already provides such an interface in Function<T,R> from the java.util.function package.

仔细看，将会发现Foo只不过是一个方法，接受一个参数，生产一个结果。 Java 8 已经提供了这样一个接口在Function<T,R> 在java.util.function包中。  

Now we can remove interface Foo completely and change our code to:  

现在我们可以移除掉interface Foo, 把代码改成这样：

```

	public String add(String string, Function<String, String> fn) {
    	return fn.apply(string);
	}
```
To execute this, we can write:

```
Function<String, String> fn = 
  parameter -> parameter + " from lambda";
String result = useFoo.add("Message ", fn);
```

## 3. Use the @FunctionalInterface Annotation 

Annotate your functional interfaces with @FunctionalInterface. At first, this annotation seems to be useless. Even without it, your interface will be treated as functional as long as it has just one abstract method.  

用@FunctionalInterface注解你自己的函数式接口。 首先，这个注解看起来是没有用的。 即使没有它，你的接口也会被视为函数，只要这个接口有一个抽象方法。  

But imagine a big project with several interfaces – it’s hard to control everything manually. An interface, which was designed to be functional, could accidentally be changed by adding of other abstract method/methods, rendering it unusable as a functional interface.  

但是想象一个大的项目有几个接口，很难用手控制每一件事。 一个被设计成函数式的接口，可能意外的被改变通过加入其它的抽象方法，使它不是一个函数式接口。

But using the @FunctionalInterface annotation, the compiler will trigger an error in response to any attempt to break the predefined structure of a functional interface. It is also a very handy tool to make your application architecture easier to understand for other developers.  

但是使用@FunctionalInterface注解，编译器将会触发一个错误，如果有任何想要打破一个函数式接口预定义结构的尝试。 这也是个非常方便的工具来使你的应用架构对其他开发者容易理解。  

So, use this:  

```

	@FunctionalInterface
	public interface Foo {
    	String method();
	}
```

instead of just: 

替代下边的： 

```
	public interface Foo {
    	String method();
	}
```

## 4. Don’t Overuse Default Methods in Functional Interfaces不要过度使用默认方法在函数式接口中  

You can easily add default methods to the functional interface. This is acceptable to the functional interface contract as long as there is only one abstract method declaration:  

你可以简单的添加默认方法到函数式接口中。 这是可接受的对于函数式接口约定，只要这里有一个抽象方法。  

```

	@FunctionalInterface
	public interface Foo {
    	String method();
    	default void defaultMethod() {}
	}
```

Functional interfaces can be extended by other functional interfaces if their abstract methods have the same signature. For example:  

函数式接口可以被拓展通过其他函数式接口，如果它们的抽象方法有相同的签名，比如：

```

	@FunctionalInterface
	public interface FooExtended extends Baz, Bar {}
     
	@FunctionalInterface
	public interface Baz {  
    	String method();    
    	default void defaultBaz() {}        
	}
     
	@FunctionalInterface
	public interface Bar {  
    	String method();    
    	default void defaultBar() {}    
	}
```  

Just as with regular interfaces, extending different functional interfaces with the same default method can be problematic. For example, assume that interfaces Bar and Baz both have a default method defaultCommon(). In this case, you will get a compile-time error:  

像普通的接口一样，拓展不同的函数式接口带有相同的默认方法，可能会是有问题的。比如，假设接口 Bar和Baz,都有默认的方法defaultCommons(). 在这个例子中，你将会看到一个编译时错误：  

```
	interface Foo inherits unrelated defaults for defaultCommon() from types Baz and Bar...
```

To fix this, defaultCommon() method should be overridden in the Foo interface. You can, of course, provide a custom implementation of this method. But if you want to use one of the parent interfaces’ implementations (for example, from the Baz interface), add following line of code to the defaultCommon() method’s body:

为修复这个问题，defaultCommon()方法应该被覆盖在Foo接口中。 当然你可以提供一个这个方法的自定义实现。但是如果你还想使用父级接口的这个实现（比如，Baz接口），把下边这一行代码加入到defaultCommon()的方法体。  

```

	Baz.super.defaultCommon();
```
But be careful. Adding too many default methods to the interface is not a very good architectural decision. It is should be viewed as a compromise, only to be used when required, for upgrading existing interfaces without breaking backward compatibility.  

但是要小心一点。添加太多的默认方法到接口中不是一个非常好的架构决定。这会被视为一个妥协，只有在必须的时候才会用，用在升级现有的接口而不破坏向后兼容。  

## 5. Instantiate Functional Interfaces with Lambda Expressions 用lambda表达式实例化函数式接口  

The compiler will allow you to use an inner class to instantiate a functional interface. However, this can lead to very verbose code. You should prefer lambda expressions:  

编译器将会允许你使用内部类来实例化一个函数式接口。然而，这会导致非常啰嗦的代码，你应该首选lambda表达式：  

```

	Foo foo = parameter -> parameter + " from Foo";
```

over an inner class: 

通过一个内部类实现：

```
Foo fooByIC = new Foo() {
    @Override
    public String method(String string) {
        return string + " from Foo";
    }
};
```

The lambda expression approach can be used for any suitable interface from old libraries. It is usable for interfaces like Runnable, Comparator, and so on. However, this doesn’t mean that you should review your whole older codebase and change everything.  

lambda表达式方法可以被用来对于任何老的库中适合的接口。 对于这些接口lambda表达式是可用的，比如，Runnable，Comparator,等等。 然而，这并不意味着你要检查你的整个老的代码库，去改掉所有的。 

## 6. Avoid Overloading Methods with Functional Interfaces as Parameters 避免过度使用函数式接口方法作为参数  

Use methods with different names to avoid collisions; let’s look at an example:  

用不同的名字使用方法来避免冲突；看一下例子：

```

	public interface Adder {
	    String add(Function<String, String> f);
	    void add(Consumer<Integer> f);
	}
	 
	public class AdderImpl implements Adder {
	 
	    @Override
	    public  String add(Function<String, String> f) {
	        return f.apply("Something ");
	    }
	 
	    @Override
	    public void add(Consumer<Integer> f) {}
	}
``` 

At first glance, this seems reasonable. But any attempt to execute any of AdderImpl’s methods: 

看第一眼，这看起来合理的。但是任何尝试执行任何AdderImpl’s 方法:  

```
String r = adderImpl.add(a -> a + " from lambda");
```

ends with an error with the following message:  

就会出现下边的错误信息：  

```
reference to add is ambiguous both method 
add(java.util.function.Function<java.lang.String,java.lang.String>) 
in fiandlambdas.AdderImpl and method 
add(java.util.function.Consumer<java.lang.Integer>) 
in fiandlambdas.AdderImpl match
```
To solve this problem, you have two options. The first is to use methods with different names:  

为解决这个问题，你有两个选择。第一个是使用方法的时候用不同的名字：  

```
String addWithFunction(Function<String, String> f);
 
void addWithConsumer(Consumer<Integer> f);
```  

The second is to perform casting manually. This is not preferred.  

第二个方法是手动执行一个类型强转。 这个不是首选。  

```
String r = Adder.add((Function) a -> a + " from lambda");
```

## 7. Don’t Treat Lambda Expressions as Inner Classes 不要对待Lambda表达式作为内部类  

Despite our previous example, where we essentially substituted inner class by a lambda expression, the two concepts are different in an important way: scope.  

尽管在前边的例子中，我们基本上用lambda表达式取代了内部类，这两个概念在一个重要的方法 scpoe作用域 上，是不同的。  

When you use an inner class, it creates a new scope. You can overwrite local variables from the enclosing scope by instantiating new local variables with the same names. You can also use the keyword this inside your inner class as a reference to its instance.  

当你使用一个内部类，它创建了一个新的作用域。你可以重写本地变量在封闭的作用域中，通过实例化新的本地变量用相同的名字。 你也可以使用关键字this,在你的内部类中，作为它实例的引用。  

However, lambda expressions work with enclosing scope. You can’t overwrite variables from the enclosing scope inside the lambda’s body. In this case, the keyword this is a reference to an enclosing instance.  

然而，lambda表达式也在封闭的作用域中工作。但是你不能在lambda表达式体中的封闭作用域重写变量。在这个情况下，关键字this是一个指向一个封闭实体的引用（外边的实体）。  

For example, in the class UseFoo you have an instance variable value:  

例如，在类UseFoo中，你有一个实例变量值： 

```
private String value = "Enclosing scope value";
```

Then in some method of this class place the following code and execute this method.  

然后在这个类的一些方法中放入下边的代码，并且执行这个方法。  

```

	public String scopeExperiment() {
	    Foo fooIC = new Foo() {
	        String value = "Inner class value";
	 
	        @Override
	        public String method(String string) {
	            return this.value;
	        }
	    };
	    String resultIC = fooIC.method("");
	 
	    Foo fooLambda = parameter -> {
	        String value = "Lambda value";
	        return this.value;
	    };
	    String resultLambda = fooLambda.method("");
	 
	    return "Results: resultIC = " + resultIC + 
	      ", resultLambda = " + resultLambda;
	}
```

If you execute the scopeExperiment() method, you will get the following result: Results: resultIC = Inner class value, resultLambda = Enclosing scope value  

如果执行scopeExperiment()方法，你将会获取下边的结果：Results: resultIC = Inner class value, resultLambda = Enclosing scope value

As you can see, by calling this.value in IC,  you can access a local variable from its instance. But in the case of the lambda, this.value call gives you access to the variable value which is defined in the  UseFoo class, but not to the variable value defined inside the lambda’s body.  

正如你看到的，通过调用this.value在IC中，你可以访问一个本地变量在它自己的实例中。但是在lambda的例子中，this.value调用，访问到了定义在UseFoo类中的变量，但是不是定义在lambda表达式体中的变量。

## 8. Keep Lambda Expressions Short And Self-explanatory 保持Lambda表达式简短并且自解释。  

If possible, use one line constructions instead of a large block of code. Remember lambdas should be an expression, not a narrative. Despite its concise syntax, lambdas should precisely express the functionality they provide.  

如果可能，使用一行构造，而不是一大块代码。记住lambdas应该是一个表达式，而不是一个讲述。尽管它的语法简洁，lambdas应该精确的表达它们提供的功能。

This is mainly stylistic advice, as performance will not change drastically. In general, however, it is much easier to understand and to work with such code.  

这主要是格式上的建议，但是性能是不会有大的改变的。一般而言，然而，用这种格式的代码更容易理解。  

This can be achieved in many ways – let’s have a closer look.

这可以用很多方法实现，仔细看一下。  

### 8.1. Avoid Blocks of Code in Lambda’s Body 避免Lambda表达式体中代码块  

In an ideal situation, lambdas should be written in one line of code. With this approach, the lambda is a self-explanatory construction, which declares what action should be executed with what data (in the case of lambdas with parameters).  

在一个理想的情况下，lambdas应该被呗写成一行。用这个方法，lambda是一个自解释构造，声明了什么动作应该被执行，带着什么数据（在lambda带参数的情况下）。

If you have a large block of code, the lambda’s functionality is not immediately clear.  

如果你有大块的代码块，lambda的功能不是瞬间明白的。  

With this in mind, do the following:  

记住这些，做下边这些代码： 

```

	Foo foo = parameter -> buildString(parameter);  
``` 

```

	private String buildString(String parameter) {
    	String result = "Something " + parameter;
    	//many lines of code
    	return result;
	}
```
不用下边的
```

	Foo foo = parameter -> { String result = "Something " + parameter; 
    	//many lines of code 
    	return result; 
	};
```

However, please don’t use this “one-line lambda” rule as dogma. If you have two or three lines in lambda’s definition, it may not be valuable to extract that code into another method. 

然而，不要使用这个“一行lambda”作为教条，如果你有两三行在lambda中，可能不值得提取这些代码到另外的方法。  

### 8.2. Avoid Specifying Parameter Types

A compiler in most cases is able to resolve the type of lambda parameters with the help of type inference. Therefore, adding a type to the parameters is optional and can be omitted.

编译器在大多数情况下可以解决lambda参数类型通过type接口的帮助，因此，添加一个类型在参数上是可选的并且可以被忽略。 

Do this:  

像这样做：

```

	(a, b) -> a.toLowerCase() + b.toLowerCase();
```

而不是这样：  
instead of this:  

```

	(String a, String b) -> a.toLowerCase() + b.toLowerCase();
``` 


### 8.3. Avoid Parentheses Around a Single Parameter 避免在一个参数的情况下用括号  

Lambda syntax requires parentheses only around more than one parameter or when there is no parameter at all. That is why it is safe to make your code a little bit shorter and to exclude parentheses when there is only one parameter.  

Lambda语法要求有括号只在参数不止一个的时候或者没有参数的时候。 这就是为什么，使你的代码更短一点并且当一个参数的时候不用括号是安全的。 

So, do this:
所以，这样做： 

```

	a -> a.toLowerCase();
``` 

instead of this:

而不是这样：

```

	(a) -> a.toLowerCase();
```

### 8.4.  Avoid Return Statement and Braces 避免返回声明，和使用 Braces 大括号

Braces and return statements are optional in one-line lambda bodies. This means, that they can be omitted for clarity and conciseness.  

大括号和返回声明（return关键字）是可选的，在一行lambda中。 这意味着，他们可以被省略为了清楚和简洁。  

Do this:

```

	a -> a.toLowerCase();
```

instead of this:  

```
	
	a -> {return a.toLowerCase()};
```

### 8.5. Use Method References 使用方法引用   

Very often, even in our previous examples, lambda expressions just call methods which are already implemented elsewhere. In this situation, it is very useful to use another Java 8 feature: method references.

经常情况下，即使在我们之前的例子中，lambda表达式仅仅调用了在别处实现的方法。 在这个情况下，使用Java 8另一个功能，方法引用，非常有用。  

So, the lambda expression:  

```

	a -> a.toLowerCase();
```
could be substituted by:

```

	String::toLowerCase;
```

This is not always shorter, but it makes the code more readable.  

这不总是更短，但是它使代码更可读。  

## 9. Use “Effectively Final” Variables 使用高效的Final变量  

Accessing a non-final variable inside lambda expressions will cause the compile-time error. But it doesn’t mean that you should mark every target variable as final. 

在lambda表达式中访问不是final的变量将会造成编译时异常。但是这不意味着你应该使每一个目标变量变成final的。  

According to the “effectively final“ concept, a compiler treats every variable as final, as long as it is assigned only once.  

关于高效的final概念，编译器对待每一个变量作为final,只要这个变量被赋值过一次。   

It is safe to use such variables inside lambdas because the compiler will control their state and trigger a compile-time error immediately after any attempt to change them.  

在lambda中使用这样的变量是安全的，因为编译器将会控制它们的状态，当试图改变它们的时候，触发一个编译时异常。  

For example, the following code will not compile:  

例如，下边的代码将不会编译通过： 

```

	public void method() {
	    String localVariable = "Local";
	    Foo foo = parameter -> {
	        String localVariable = parameter;
	        return localVariable;
	    };
	}
```  

The compiler will inform you that: 

编译器将会通知：

```

	Variable 'localVariable' is already defined in the scope.
``` 

This approach should simplify the process of making lambda execution thread-safe.  

这个方法应该简化了使lambda执行线程安全的处理。  

不知道为什么说了半天final,举了这么个例子。  

## 10. Protect Object Variables from Mutation 保护Object变量不受Mutation突变损害

One of the main purposes of lambdas is use in parallel computing – which means that they’re really helpful when it comes to thread-safety.  

Lambdas的几大主要用途之一是在并行计算中，这意味着Lambda在线程安全上非常有帮助。 

The “effectively final” paradigm helps a lot here, but not in every case. Lambdas can’t change a value of an object from enclosing scope. But in the case of mutable object variables, a state could be changed inside lambda expressions.  

这个高效的final范例在这有很大帮助，但是不是在每种例子中。Lambda不能改变外边类里边的变量（enclosing scope）。但是在mutable object变量的情况中，一个声明state可以在lambda表达式中改变。  

Consider the following code: 

考虑下边的代码：

```

	int[] total = new int[1];
	Runnable r = () -> total[0]++;
	r.run();
```

This code is legal, as total variable remains “effectively final”. But will the object it references to have the same state after execution of the lambda? No!  

这个代码是正常的，正如所有的变量保持高效的final,但是这里引用的对象在lambda执行过之后还有相同的state吗？不会。

Keep this example as a reminder to avoid code that can cause unexpected mutations.  

把这个例子作为一个提醒，来避免会造成意外的mutations突变，变化的代码。

## 11. Conclusion 总结

In this tutorial, we saw some best practices and pitfalls in Java 8’s lambda expressions and functional interfaces. Despite the utility and power of these new features, they are just tools. Every developer should pay attention while using them.

在这个教程中，我们看到一些很好的实践和陷阱在Java 8的Lambda表达式和函数式接口。尽管实用性和这些新功能的强大，但是它们仅仅是工具。每个开发者应该集中注意力当使用的时候。

The complete source code for the example is available in this GitHub project – this is a Maven and Eclipse project, so it can be imported and used as-is. 

*** 
Reference

[difference-between-mutable-objects-and-immutable-objects](https://stackoverflow.com/questions/4658453/difference-between-mutable-objects-and-immutable-objects)








