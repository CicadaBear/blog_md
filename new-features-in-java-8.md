# New Features in Java 8

From [java-8-new-features](http://www.baeldung.com/java-8-new-features)

## 1. Overview 概览

In this article, we’ll have a quick look at some of the most interesting new features in Java 8.  

这篇文章，我们将快速看一下Java 8中一些最有意思的新功能。

We’ll talk about: interface default and static methods, method reference and Optional. 

我们将要说说，interface默认的和静态的方法，method reference方法引用，和Optional。

We have already covered some the features of the Java 8’s release – [stream API](http://www.baeldung.com/java-8-streams-introduction), [lambda expressions and functional interfaces](http://www.baeldung.com/java-8-lambda-expressions-tips) – as they’re comprehensive topics that deserve a separate look.  

我们已经涉及了一些Java 8的新功能， [stream API](http://www.baeldung.com/java-8-streams-introduction), [lambda expressions and functional interfaces](http://www.baeldung.com/java-8-lambda-expressions-tips)，它们都是综合的主题，值得分开一看。  

## 2. Interface Default and Static Methods 接口默认静态方法 

Before Java 8, interfaces could have only public abstract methods. It was not possible to add new functionality to the existing interface without forcing all implementing classes to create an implementation of the new methods, nor it was possible to create interface methods with an implementation.  

Java 8之前，接口只能有public 抽象方法。 在现有的接口添加新的方法而不强迫所有的实现类实现新的方法是不可能的。创建一个带不抽象方法的接口是不可能的。  

Starting with Java 8, interfaces can have static and default methods that, despite being declared in an interface, have a defined behavior.  

从Java 8开始，接口可以有静态默认方法，尽管定义在接口中，但是有一个确定的行为。  

### 2.1. Static Method 静态方法 

Consider the following method of the interface (let’s call this interface Vehicle):  

考虑下边的接口方法，暂且叫这个接口Vehicle：

```
static String producer() {
    return "N&F Vehicles";
}
```  

The static producer() method is available only through and inside of an interface. It can’t be overridden by an implementing class.  

这个静态方法producer()，只能通过这个接口，或者在这个接口内可用。 这个方法不能被实现类覆盖。 

To call it outside the interface the standard approach for static method call should be used: 

想要在接口外调用的这个方法，标准做法是：

```
String producer = Vehicle.producer();
```  

### 2.2. Default Method 

Default methods are declared using the new default keyword. These are accessible through the instance of the implementing class and can be overridden.  

默认方法声明用default关键字。 这些是可以通过实现类的实例来访问的而且可以被覆盖。  

Let’s add a default method to our Vehicle interface, which will also make a call to the static method of this interface:  

给Vehicle接口加个默认方法，在这个方法里边调用这个接口的静态方法:  

```
default String getOverview() {
    return "ATV made by " + producer();
}
```  

Assume that this interface is implemented by the class VehicleImpl. For executing the default method an instance of this class should be created:  

假设这个接口被VehicleImpl实现类实现。 为了执行默认方法，需要创建一个这个类的实例：  

```

	Vehicle vehicle = new VehicleImpl();
	String overview = vehicle.getOverview();
```

## 3. Method References 方法引用  

Method reference can be used as a shorter and more readable alternative for a lambda expression which only calls an existing method. There are four variants of method references.  

方法引用可以被用来做为一个更短的更可读的lambda表达式的替代，方法引用只调用存在的方法。 这里有4种方法引用。  

### 3.1. Reference to a Static Method 引用一个静态方法  

The reference to a static method holds the following syntax: ContainingClass::methodName.   

静态方法的引用支持下边的语法：ContainingClass::methodName。  

Let’s try to count all empty strings in the List<String> with help of Stream API.   

尝试计算List<String>中所有的空字符串，用Stream API.  

```
boolean isReal = list.stream().anyMatch(u -> User.isRealUser(u));
```  

Take a closer look at lambda expression in the filter() method, it just makes a call to a static method isRealUser(User user) of the User class. So it can be substituted with a reference to a static method:  

仔细看看filter()中的lambda表达式，它调用了User类的静态方法isRealUser（User user)。所以中可以被方法引用替代：

```
boolean isReal = list.stream().anyMatch(User::isRealUser);
```  
This type of code looks much more informative.  

这样看起来更可读清晰，可告知性。  

### 3.2. Reference to an Instance Method 实例方法引用  

The reference to an instance method holds the following syntax: containingInstance::methodName. Following code calls method isLegalName(String string) of type User which validates an input parameter:  

实例方法引用支持以下语法：containingInstance::methodName. 下边的代码调用了方法User类型的isLegalName(String string)，这个方法用来验证输入参数：  

```
	
	User user = new User();
	boolean isLegalName = list.stream().anyMatch(user::isLegalName);
```

### 3.3. Reference to an Instance Method of an Object of a Particular Type 一种特定类型的方法引用 

This reference method takes the following syntax: ContainingType::methodName. An example:   

这个方法引用采用下边的语法：ContainingType::methodName，例子：

```

	long count = list.stream().filter(String::isEmpty).count();
```  

### 3.4. Reference to a Constructor 构造方法引用  

A reference to a constructor takes the following syntax: ClassName::new. As constructor in Java is a special method, method reference could be applied to it too with the help of new as a method name.  

一个构造方法引用采用下边的语法：ClassName::new。 因为构造方法在Java中是特殊的方法，方法引用当然也可以被应用，用new作为方法名。 


```

	Stream<User> stream = list.stream().map(User::new);
``` 

## 4. Optional<T>  

Before Java 8 developers had to carefully validate values they referred to, because of a possibility of throwing the NullPointerException (NPE). All these checks demanded a pretty annoying and error-prone boilerplate code.   

Java 8之前的开发者必须仔细的验证他们引用的值，因为有抛空指针异常的可能。 所有的这些检查需要一个非常恼人的，易于出错的样板代码。  

Java 8 Optional<T> class can help to handle situations where there is a possibility of getting the NPE. It works as a container for the object of type T. It can return a value of this object if this value is not a null. When the value inside this container is null it allows doing some predefined actions instead of throwing NPE.  

Java 8的Optional<T>类可以帮助解决这种情况，什么情况，有可能出现空指针异常的情况。 它以一个T类型对象的容器来工作。 它可以返回一个这个对象的值，如果值不为空。当容器里边的值是空，允许做一些预定义的动作而不是抛空指针异常。  

### 4.1. Creation of the Optional<T> 创建Optional<T>  

An instance of the Optional class can be created with the help of its static methods:  

用Optional的静态方法创建：  

```
Optional<String> optional = Optional.empty();
```  
Returns an empty Optional.  
返回一个空Optional。

```
String str = "value";
Optional<String> optional = Optional.of(str);
``` 
Returns an Optional which contains a non-null value.  
返回一个空Optional，包含一个非空值。   

```
Optional<String> optional = Optional.ofNullable(getString());
```  
Will return an Optional with a specific value or an empty Optional if the parameter is null.  
将会返回一个带有特定值的Optional,或者空Optional如果参数为空。  

### 4.2. Optional<T> usage  Optional用法  

For example, you expect to get a List<String> and in the case of null you want to substitute it with a new instance of an ArrayList<String>. With pre-Java 8’s code you need to do something like this:  

例如，你期望获取一个List<String>，并且如果是null的时候，你想用一个new ArrayList<String>类替代。 Java 8之前的代码，需要这样做：  

```
	
	List<String> list = getList();
	List<String> listOpt = list != null ? list : new ArrayList<>();
```  

With Java 8 the same functionality can be achieved with a much shorter code:  

Java 8相同的功能可以用更短的代码实现：  

```
	List<String> listOpt = getList().orElseGet(() -> new ArrayList<>());
```  

There is even more boilerplate code when you need to reach some object’s field in the old way. Assume you have an object of type User which has a field of type Address with a field street of type String. And for some reason you need to return a value of the street field if some exist or a default value if street is null:  

这里还有更多的模板代码，当你需要获取一些对象的属性时，用老方法。假设你有一个User类型的对象，有一个Address类型的属性，Address类型有一个字符串类型的street属性。 现在有一些原因你需要返回street的值，如果存在，如果不存在返回一个默认值。  

```
	
	User user = getUser();
	if (user != null) {
	    Address address = user.getAddress();
	    if (address != null) {
	        String street = address.getStreet();
	        if (street != null) {
	            return street;
	        }
	    }
	}
	return "not specified";
```  

This can be simplified with Optional:  
这个可以用Optional简化：  

```

	Optional<User> user = Optional.ofNullable(getUser());
	String result = user
  	.map(User::getAddress)
  	.map(Address::getStreet)
  	.orElse("not specified");
```  

In this example we used the map() method to convert results of calling the getAdress() to the Optional\<Address\> and getStreet() to Optional<String>. If any of these methods returned null the map() method would return an empty Optional.  

在这个例子中，我们使用map()方法来转化调用getAddress()的返回结果成Optional\<Address\>,还有getStreet()转化成Optional<String>。如果这里任何方法返回null,map()方法将返回一个empty的Optional。  


Imagine that our getters return Optional\<T\>. So, we should use the flatMap() method instead of the map():  

想象,我们的getters方法返回Optional<T>。所以我们应该使用flatMap()方法而不是map():  

```
	
	Optional<OptionalUser> optionalUser = Optional.ofNullable(getOptionalUser());
	String result = optionalUser
  		.flatMap(OptionalUser::getAddress)
  		.flatMap(OptionalAddress::getStreet)
  		.orElse("not specified");
```  

Another use case of Optional is changing NPE with another exception. So, as we did previously, let’s try to do this in pre-Java 8’s style:  

另一个Optional的用例是把空指针异常转化成其他的异常。先看，Java 8之前的风格：  

```

	String value = null;
	String result = "";
	try {
	    result = value.toUpperCase();
	} catch (NullPointerException exception) {
	    throw new CustomException();
	}
```  

And what if we use Optional\<String\>? The answer is more readable and simpler:  

如果我们使用 Optional\<String\>会怎么样？更可读，简单：  

```

	String value = null;
	Optional<String> valueOpt = Optional.ofNullable(value);
	String result = valueOpt.orElseThrow(CustomException::new).toUpperCase();
```

Notice, that how and for what purpose to use Optional in your app is a serious and controversial design decision, and explanation of its all pros and cons is out of the scope of this article. If you are interested, you can dig deeper, there are plenty of interesting articles on the Internet devoted to this problem. [This one](http://blog.joda.org/2014/11/optional-in-java-se-8.html) and [this another](http://blog.joda.org/2015/08/java-se-8-optional-pragmatic-approach.html) one could be very helpful.  

注意，怎样和为什么目的使用Optional是一个严肃的有争议的设计决定，并且解释它所有的优缺点超出了本篇文章的范围。如果你感兴趣，你可以挖的更深，网上有很多的有趣的文章关于这个问题。这两篇文章可能非常有帮助。 

## 5. Conclusion 总结  

In this article, we are briefly discussing some interesting new features in Java 8.  

在这篇文章中，我们简略地讨论了一些Java 8中有趣的功能。  

There are of course many other additions and improvements which are spread across many Java 8 JDK packages and classes.  

Java 8 JDK很多包和classes中都有所增加和提升。  

But, the information illustrated in this article is a good starting point for exploring and learning about some of these new features.  

但是，这篇文章中阐述的信息是一个探索学习这些新功能的好的开端

Finally, all the source code for the article is available on [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java).  
















