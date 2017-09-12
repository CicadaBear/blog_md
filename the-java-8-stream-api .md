#The Java 8 Stream API 

From [java-8-streams](http://www.baeldung.com/java-8-streams)

## 1. Overview 概览  

In this in-depth tutorial, we will go through the practical usage of Java 8 Streams from creation to parallel execution.  

在这个深入的教程中，我们将要通过实际的使用，从Stream的创建到并行执行。  

To understand this material reader should have the basic knowledge of Java 8 (lambda expressions, Optional, method references) and Stream API. If you aren’t familiar with these topics, please take a look at our previous articles – [New Features in Java 8](http://www.baeldung.com/java-8-new-features) and [Introduction to Java 8 Streams](http://www.baeldung.com/java-8-streams-introduction).  

为理解这篇材料，读者需要有Java 8的基础知识（lambda expressions, Optional, method references）和Stream API。如果你对这些话题不熟悉，看一下之前的文章， [New Features in Java 8](http://www.baeldung.com/java-8-new-features) and [Introduction to Java 8 Streams](http://www.baeldung.com/java-8-streams-introduction).   


## 2. Stream Creation 创建流  

There are many ways to create a stream instance of different sources. Once created, the instance will not modify its source, therefore allowing the creation of multiple instances from a single source.  

这里有很多方法来创建多种不同源数据的实例。 一旦创建，这个实例将不会修改它的源数据，因此允许用一个数据源来创建多个实例。  

### 2.1. Empty Stream 

The empty() method should be used in case of a creation of an empty stream:  

```
Stream<String> streamEmpty = Stream.empty();
```  

Its often the case that the empty() method is used upon creation to avoid returning null for streams with no element:  

empty()方法经常被用来避免返回null。


```
public Stream<String> streamOf(List<String> list) {
    return list == null || list.isEmpty() ? Stream.empty() : list.stream();
}
```
 
### 2.2. Stream of Collection  

Stream can also be created of any type of Collection (Collection, List, Set):  

Stream可以被任何集合创建。

```
	
	Collection<String> collection = Arrays.asList("a", "b", "c");
	Stream<String> streamOfCollection = collection.stream();
```  

### 2.3. Stream of Array
 
Array can also be a source of a Stream:  

```
Stream<String> streamOfArray = Stream.of("a", "b", "c");
```

They can also be created out of an existing array or of a part of an array:  

它们可以用一个存在的数组创建，或者数组的一部分。  

```
String[] arr = new String[]{"a", "b", "c"};
Stream<String> streamOfArrayFull = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);
```

### 2.4. Stream.builder()

When builder is used the desired type should be additionally specified in the right part of the statement, otherwise the build() method will create an instance of the Stream<Object>:  

当builder使用的时候，想要的类型应该被指定在声明的右边 泛型，否则build方法将会创建一个Stream<Object>的实例：  

```
Stream<String> streamBuilder =
  Stream.<String>builder().add("a").add("b").add("c").build();
```
### 2.5. Stream.generate()   

The generate() method accepts a Supplier<T> for element generation. As the resulting stream is infinite, developer should specify the desired size or the generate() method will work until it reaches the memory limit:   

generate()方法接受一个Supplier<T>为element生成。 当结果stream是无限大的时候，开发者应该指定一个想要的数量否则generate()方法就会直到内存用完。  

```
Stream<String> streamGenerated =
  Stream.generate(() -> "element").limit(10);
``` 
The code above creates a sequence of ten strings with the value – “element”.   

上边的代码创建了一个10个string “element”的序列。 

### 2.6. Stream.iterate() 

Another way of creating an infinite stream is by using the iterate() method:   

另一个创建无限stream的方法是使用iterate()方法：  

```
Stream<Integer> streamIterated = Stream.iterate(40, n -> n + 2).limit(20);
```

The first element of the resulting stream is a first parameter of the iterate() method. For creating every following element the specified function is applied to the previous element. In the example above the second element will be 42.  

结果stream中第一个的element是iterate()方法的第一个参数。为创建接下来的每一个element,这个指定的方法被应用于前一个element. 在这个例子中，第二个element将会是42。  

### 2.7. Stream of Primitives 基本类型Stream  

Java 8 offers a possibility to create streams out of three primitive types: int, long and double. As Stream<T> is a generic interface and there is no way to use primitives as a type parameter with generics, three new special interfaces were created: IntStream, LongStream, DoubleStream.  

Java 8 提供了从三种基本类型int,long,double创建stream的可能。 因为Stream<T> 是一个泛型接口，这里没有方法来使用基本数据类型作为泛型的参数，3个新的特殊的接口准备好了，IntStream, LongStream, DoubleStream.

Using the new interfaces alleviates unnecessary auto-boxing allows increased productivity:  

使用新的接口减轻了不必要的自动包装，提高了生产率。 

```
IntStream intStream = IntStream.range(1, 3);
LongStream longStream = LongStream.rangeClosed(1, 3);
```
The range(int startInclusive, int endExclusive) method creates an ordered stream from the first parameter to the second parameter. It increments the value of subsequent elements with the step equal to 1. The result doesn’t include the last parameter, it is just an upper bound of the sequence.  

range(int startInclusive, int endExclusive)方法创建一个有序的stream从第一个参数到第二个参数。它增加子序列的element值，一步加一。 这个结果不包括最后一个参数，它仅仅是序列的上界。  

The rangeClosed(int startInclusive, int endInclusive) method does the same with only one difference – the second element is included. These two methods can be used to generate any of the three types of streams of primitives.  

rangeClosed(int startInclusive, int endInclusive) 方法和之前的一样只有一个不同的，第二个element是包含的。 这两个方法可以被用来生成任何三种基本类型变量的stream。

Since Java 8 the Random class provides a wide range of methods for generation streams of primitives. For example, the following code creates a DoubleStream, which has three elements:  

Java 8的Random类提供了很多方法来生成基本数据类型的stream。例如，下边的代码创建一个有三个element的DoubleStream。  

```
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);
```

### 2.8. Stream of String

String can also be used as a source for creating a stream.  

String 也可以被用来作为一个源来创建stream。 

With the help of the chars() method of the String class. Since there is no interface CharStream in JDK, the IntStream is used to represent a stream of chars instead.  

使用String的chars()方法。既然JDK没有CharStream接口，IntStream被用来代表  

```
IntStream streamOfChars = "abc".chars();
```

The following example breaks a String into sub-strings according to specified RegEx:  

```
Stream<String> streamOfString =
  Pattern.compile(", ").splitAsStream("a, b, c");
```

### 2.9. Stream of File

Java NIO class Files allows to generate a Stream<String> of a text file through the lines() method. Every line of the text becomes an element of the stream:  

Java NIO 的class类允许用文本文件生成一个Stream<String>通过lines()方法。每一行文本变成一个stream的element。  

```
Path path = Paths.get("C:\\file.txt");
Stream<String> streamOfStrings = Files.lines(path);
Stream<String> streamWithCharset = 
  Files.lines(path, Charset.forName("UTF-8"));
```  

The Charset can be specified as an argument of the lines() method.  

编码可以作为lines()的参数指定。  

## 3. Referencing a Stream  引用一个Stream

It is possible to instantiate a stream and to have an accessible reference to it as long as only intermediate operations were called. Executing a terminal operation makes a stream inaccessible.  

只要中间操作被调用，实例化一个stream并且有一个可以访问的引用是可以实现。 执行一个最终操作使一个stream不可访问。  

To demonstrate this we will forget for a while that the best practice is to chain sequence of operation. Besides its unnecessary verbosity, technically the following code is valid:  

为演示这个我们将忘记一会儿，“最好的实践是链式操作序列”。除了不必要的啰嗦，在技术上，下边的代码是合法的：  

```
Stream<String> stream = 
  Stream.of("a", "b", "c").filter(element -> element.contains("b"));
Optional<String> anyElement = stream.findAny();
```

But an attempt to reuse the same reference after calling the terminal operation will trigger the IllegalStateException:  

但是在调用最终操作后尝试重用相同的引用会触发一个IllegalStateException：  

```
Optional<String> firstElement = stream.findFirst();
```  

As the IllegalStateException is a RuntimeException, a compiler will not signalize about a problem. So, it is very important to remember that Java 8 streams can’t be reused.   

因为IllegalStateException是RuntimeException，编译器不会对这个问题发信号。所以，记住Java 8 stream不能被重用非常重要。  

This kind of behavior is logical because streams were designed to provide an ability to apply a finite sequence of operations to the source of elements in a functional style, but not to store elements.   

这种行为是有逻辑的，因为stream被设计用来为一个多元素数据源提供一个有限的序列操作，以一种函数式的风格，但是不存储元素。  

So, to make previous code work properly some changes should be done:  

所以，想要使之前的代码恰当的工作，我们需要做一些改动:  

```

	List<String> elements =
  	Stream.of("a", "b", "c").filter(element -> element.contains("b"))
    .collect(Collectors.toList());
	Optional<String> anyElement = elements.stream().findAny();
	Optional<String> firstElement = elements.stream().findFirst();
```

## 4. Stream Pipeline Stream管道  

To perform a sequence of operations over the elements of the data source and aggregate their results, three parts are needed – the source, intermediate operation(s) and a terminal operation.   

为在一个数据源的元素上执行一个操作序列，并且聚合它们的结果，三个部分是必须的，数据源，中间操作，最终操作。  

Intermediate operations return a new modified stream. For example, to create a new stream of the existing one without few elements the skip() method should be used:  

中间操作返回一个新的修改过的stream.例如，为用一个现有的stream创建一个新的stream,剔除其中几个元素，可以用skip()方法:  

```

	Stream<String> onceModifiedStream =
  	Stream.of("abcd", "bbcd", "cbcd").skip(1);
```

If more than one modification is needed, intermediate operations can be chained. Assume that we also need to substitute every element of current Stream<String> with a sub-string of first few chars. This will be done by chaining the skip() and the map() methods:  

如果需要不止一个修改，中间操作可以是链式的。假设我们也需要把当前stream中的每个element替换成前边的几个字符。 这个可以用skip()和map()链式实现:  

```
	Stream<String> twiceModifiedStream =
  	stream.skip(1).map(element -> element.substring(0, 3));
```

As you can see, the map() method takes a lambda expression as a parameter. If you want to learn more about lambdas take a look at our tutorial [Lambda Expressions and Functional Interfaces: Tips and Best Practices](http://www.baeldung.com/java-8-lambda-expressions-tips).  

正如你看到的，map()方法将lambda表达式作为一个参数。如果想要学习更多关于lambdas的，看一下下边这个教程。  

A stream by itself is worthless, the real thing a user is interested in is a result of the terminal operation, which can be a value of some type or an action applied to every element of the stream. Only one terminal operation can be used per stream.  

一个stream自己没有任何价值，用户真正关心的是最终操作之后的结果，结果可能是一个类型的值或者是一个应用到这个stream的每一个element的动作。（这段话没有什么东西，但是value跟action不通啊）每个stream只有一个最终操作。  

The right and most convenient way to use streams are by a stream pipeline, which is a chain of stream source, intermediate operations, and a terminal operation. For example:  

正确的和最方便的方法来使用stream是stream管道，这个方法是stream 源，中间操作，最终操作的一个链。 例如：  

```
	
	List<String> list = Arrays.asList("abc1", "abc2", "abc3");
	long size = list.stream().skip(1)
  	.map(element -> element.substring(0, 3)).sorted().count();
```

## 5. Lazy Invocation 懒调用  

Intermediate operations are lazy. This means that they will be invoked only if it is necessary for the terminal operation execution.  

中间操作是懒操作。 这个意思是，中间操作只有在最终操作必须的时候才被调用。  

To demonstrate this, imagine that we have method wasCalled(), which increments an inner counter every time it was called:  

为了演示这个，想象我们有个方法叫wasCalled（），每一次被调用内部计数器加一：  

```
	
	private long counter;
  
	private void wasCalled() {
	    counter++;
	}
```

Let’s call method wasCalled() from operation filter():  

我们在filter()中调用了wasCalled()：  

```

	List<String> list = Arrays.asList(“abc1”,“abc2”,“abc3”);
	counter = 0;
	Stream<String> stream = list.stream().filter(element -> {
	    wasCalled();
	    return element.contains("2");
	});
```

As we have a source of three elements we can assume that method filter() will be called three times and the value of the counter variable will be 3. But running this code doesn’t change counter at all, it is still zero, so, the filter() method wasn’t called even once. The reason why – is missing of the terminal operation.   

我们有一个三个元素的数据源，我们可以假设filter()将会被调用3次并且计数器的值将会是3。但是运行这段代码并不会改变计数器的值，它仍然是0，所以filter()方法一次也没有调用。原因是，缺少了最终操作。  

Let’s rewrite this code a little bit by adding a map() operation and a terminal operation – findFirst(). We will also add an ability to track an order of method calls with a help of logging:  

让我们重写这段代码，加上个map()操作和最终操作-findFirst().我们将会有能力追踪方法的调用顺序，在日志的帮助下：  

```
	
	Optional<String> stream = list.stream().filter(element -> {
    	log.info("filter() was called");
    	return element.contains("2");
		}).map(element -> {
    log.info("map() was called");
    return element.toUpperCase();
	}).findFirst();
```

Resulting log shows that the filter() method was called twice and the map() method just once. It is so because the pipeline executes vertically. In our example the first element of the stream didn’t satisfy filter’s predicate, then the filter() method was invoked for the second element, which passed the filter. Without calling the filter() for third element we went down through pipeline to the map() method.  

结果日志显示，filter()方法调用了两次，map()方法只调用了一次。这样是因为管道是垂直执行的。在例子中，stream的第一个元素不满足filter的断言，然后filter()方法就调用了第二次，这次通过了filter. 不用调用filter()第三次，我们通过了管道进入map()方法。  

The findFirst() operation satisfies by just one element. So, in this particular example the lazy invocation allowed to avoid two method calls – one for the filter() and one for the map().  

这个findFirst()操作被一个元素满足。所以，在这个特定的例子中，懒调用允许避免两个方法调用，一个是filter()方法，一个是map()方法。  

## 6. Order of Execution 执行顺序  

From the performance point of view, the right order is one of the most important aspects of chaining operations in the stream pipeline:  

从执行的观点看，正确的执行顺序是stream管道中链式操作最重要的方面之一：  

```
	
	long size = list.stream().map(element -> {
    	wasCalled();
    	return element.substring(0, 3);
	}).skip(2).count();
```

Execution of this code will increase the value of the counter by three. This means that the map() method of the stream was called three times. But the value of the size is one. So, resulting stream has just one element and we executed the expensive map() operations for no reason twice out of three times.  

这段代码的执行将会把计数器的值加3.这意味着map()方法被调用了3次。但是size的值是1. 所以，结果stream只有一个元素并且我们白白执行了三次里边的两次。  

If we change the order of the skip() and the map() methods, the counter will increase only by one. So, the method map() will be called just once:   

如果我们改变skip()和map()的顺序，这个计数器将会只增加1.所以，这个map()方法将会只被调用一次。  

```
	
	long size = list.stream().skip(2).map(element -> {
    	wasCalled();
    	return element.substring(0, 3);
	}).count();
``` 

This brings us up to the rule: intermediate operations which reduce the size of the stream should be placed before operations which are applying to each element. So, keep such methods as skip(), filter(), distinct() at the top of your stream pipeline.   

这里提出了一个规则：减少stream大小的中间操作应该被放置在应用于每个元素的操作之前，把如下方法，放置在最前边，skip(), filter(), distinct()。  


### 7. Stream Reduction Stream减少  

The API has many terminal operations which aggregate a stream to a type or to a primitive, for example, count(), max(), min(), sum(), but these operations work according to the predefined implementation. And what if a developer needs to customize a Stream’s reduction mechanism? There are two methods which allow to do this – the reduce() and the collect() methods.  

这个API有很多的终极操作，这些操作聚合一个stream成一个类型，或者成为一个简单的，例如，count(), max(), min(), sum(), 但是这些操作对于一个预定义的实现工作。 并且如果一个开发者需要自定义一个减少Stream的机制呢？ 这里有两个方法允许做这些，reduce()方法和collect()方法。  

7.1. The reduce() Method  

There are three variations of this method, which differ by their signatures and returning types. They can have the following parameters:  

这里有3种这个方法的变形，它们的签名和返回类型不同，它们有如下参数：  

**identity** – the initial value for an accumulator or a default value if a stream is empty and there is nothing to accumulate;  

累加器的初始值或者默认值，如果一个stream是空的，这里没有东西来累加。  

**accumulator** – a function which specifies a logic of aggregation of elements. As accumulator creates a new value for every step of reducing, the quantity of new values equals to the stream’s size and only the last value is useful. This is not very good for the performance.  

累加器，是一个方法，这个方法指定了一个元素聚合的逻辑。 当累加器为每步reducing创建一个值，这个新值的数量等于这个stream的元素个数，并且只有最后一个值有用。 这里的性能表现不是很好。

**combiner** – a function which aggregates results of the accumulator. Combiner is called only in a parallel mode to reduce results of accumulators from different threads.  

组合器，是一个方法，这个方法聚合累加器的结果。 组合器只在并行模式下从不同的线程来减少累加器的结果。  

So, let’s look at these three methods in action:  

```	
	OptionalInt reduced =
  	IntStream.range(1, 4).reduce((a, b) -> a + b);
```

reduced = 6 (1 + 2 + 3) 

```
int reducedTwoParams =
  IntStream.range(1, 4).reduce(10, (a, b) -> a + b);
```
reducedTwoParams = 16 (10 + 1 + 2 + 3) 

```
int reducedParams = Stream.of(1, 2, 3)
  .reduce(10, (a, b) -> a + b, (a, b) -> {
     log.info("combiner was called");
     return a + b;
  });
```

The result will be the same as in the previous example (16) and there will be no log in which means, that combiner wasn’t called. To make a combiner work, a stream should be parallel:  

结果跟之前的例子一样并且这里没有log,这意味着，组合器没有被调用。为了让组合器工作，一个stream应该是并行的。  


```
The result will be the same as in the previous example (16) and there will be no login which means, that combiner wasn’t called. To make a combiner work, a stream should be parallel:
``` 

The result here is different (36) and the combiner was called twice. Here the reduction works by the following algorithm: accumulator ran three times by adding every element of the stream to identity to every element of the stream. These actions are being done in parallel. As a result, they have (10 + 1 = 11; 10 + 2 = 12; 10 + 3 = 13;). Now combiner can merge these three results. It needs two iterations for that (12 + 13 = 25; 25 + 11 = 36).  

这里的结果不一样了，是36并且组合器被调用了两次。这里reduction工作在下边的算法上：累加器运行三次，把每一个元素加上identity到stream的每一个元素上。这些动作以并行方式运行。作为结果，他们有(10 + 1 = 11; 10 + 2 = 12; 10 + 3 = 13;)。现在组合器器可以合并这些结果。它需要两次迭代，(12 + 13 = 25; 25 + 11 = 36).  

### 7.2. The collect() Method 

Reduction of a stream can also be executed by another terminal operation – the collect() method. It accepts an argument of the type Collector, which specifies the mechanism of reduction. There are already created predefined collectors for most common operations. They can be accessed with the help of the Collectors type.  

一个stream的减少也可以被另一个终极操作执行，就是collect()方法。 它接受一个Collector类型的参数，这个参数指定了reduction的机制。这里有已经预定义的collector为最常用的操作。它们可以通过Collectors类型来访问。  

In this section we will use the following List as a source for all streams:

在这个章节中，我们将会使用如下的List作为所有stream的数据源：

```
List<Product> productList = Arrays.asList(new Product(23, "potatoes"),
  new Product(14, "orange"), new Product(13, "lemon"),
  new Product(23, "bread"), new Product(13, "sugar"));
```

**Converting a stream to the Collection (Collection, List or Set):**  

把一个stream转换成集合(Collection, List or Set):

```
List<String> collectorCollection = 
  productList.stream().map(Product::getName).collect(Collectors.toList());
```
**Reducing to String:**  
Reducing成一个String:

```
String listToString = productList.stream().map(Product::getName)
  .collect(Collectors.joining(", ", "[", "]"));
```
The joiner() method can have from one to three parameters (delimiter, prefix, suffix). The handiest thing about using joiner() – developer doesn’t need to check if the stream reaches its end to apply the suffix and not to apply a delimiter. Collector will take care of that.  

joiner()方法可以有一二三个参数(delimiter, prefix, suffix)，分隔符，前缀，后缀。关于joiner()最方便的是，开发者不需要检查是否stream到了最后来应用后缀并且不用加分隔符。Collector会自己负责这些。  

**Processing the average value of all numeric elements of the stream:**   

处理一个stream的所有数值元素的平均值：

```
double averagePrice = productList.stream()
  .collect(Collectors.averagingInt(Product::getPrice));
```
**Processing the sum of all numeric elements of the stream:** 

处理一个stream的所有数值元素的和：

```
int summingPrice = productList.stream()
  .collect(Collectors.summingInt(Product::getPrice));
``` 

Methods averagingXX(), summingXX() and summarizingXX() can work as with primitives (int, long, double) as with their wrapper classes (Integer, Long, Double). One more powerful feature of these methods is providing the mapping. So, developer doesn’t need to use an additional map() operation before the collect() method.  

方法averagingXX(),summingXX() and summarizingXX(),这些方法可以使用基本数据类型以他们的包装类型。这些方法还有一个更强的功能是提供了mapping。 所以，开发者不需要在collect()方法之前使用附加的map()操作。 

**Collecting statistical information about stream’s elements:**  

Collecting 统计信息关于stream的元素:

```
IntSummaryStatistics statistics = productList.stream()
  .collect(Collectors.summarizingInt(Product::getPrice));
```

By using the resulting instance of type IntSummaryStatistics developer can create a statistical report by applying toString() method. The result will be a String common to this one “IntSummaryStatistics{count=5, sum=86, min=13, average=17,200000, max=23}”.   

通过使用IntSummaryStatistics的实例作为结果，开发者可以创建统计报告通过应该toString()方法。 结果将会是一个跟“IntSummaryStatistics{count=5, sum=86, min=13, average=17,200000, max=23}”相同的字符串。  

It is also easy to extract from this object separate values for count, sum, min, average by applying methods getCount(), getSum(), getMin(), getAverage(), getMax(). All these values can be extracted from a single pipeline.  

非常简单从对象各自的值中提取count,sum,min,average通过应用方法getCount(),getSum(),getMin(),getAverage(),getMax().所有这些值可以被提取出从一个管道中。  

**Grouping of stream’s elements according to the specified function:**

根据指定的方法给stream的元素分组：

```
Map<Integer, List<Product>> collectorMapOfLists = productList.stream()
  .collect(Collectors.groupingBy(Product::getPrice));
```

In the example above the stream was reduced to the Map which groups all products by their price.  

在上边的例子中，stream被reduce成Map,这个Map分组聚合了所有products通过他们的价格。  

**Dividing stream’s elements into groups according to some predicate:** 

根据一些判断分开stream的元素：  

```
Map<Boolean, List<Product>> mapPartioned = productList.stream()
  .collect(Collectors.partitioningBy(element -> element.getPrice() > 15));
```

**Pushing the collector to perform additional transformation:**  

推动collector来执行附加的转化：

```
Set<Product> unmodifiableSet = productList.stream()
  .collect(Collectors.collectingAndThen(Collectors.toSet(),
  Collections::unmodifiableSet));
```

In this particular case, the collector has converted a stream to a Set and then created the unmodifiable Set out of it.  

在这个特定的例子中，这个collector把一个stream转化成了Set,然后用这个Set创建了一个unmodifiable Set。 

**Custom collector:**  

自定义collector：

If for some reason, a custom collector should be created, the most easier and the less verbose way of doing so – is to use the method of() of the type Collector.  

如果因为某些原因，需要一个自定义的collector，最简单的最方便的方法是，使用Collector的方法of()。

```
Collector<Product, ?, LinkedList<Product>> toLinkedList =
  Collector.of(LinkedList::new, LinkedList::add, 
    (first, second) -> { 
       first.addAll(second); 
       return first; 
    });
 
LinkedList<Product> linkedListOfPersons =
  productList.stream().collect(toLinkedList);
```

In this example, an instance of the Collector got reduced to the LinkedList<Product>.  

## Conclusions

The Stream API is a powerful but simple to understand set of tools for processing sequence of elements. It allows us to reduce a huge amount of boilerplate code, create more readable programs and improve app’s productivity when used properly. 

Stream API是一个强大的但是容易理解的工具集，为处理元素序列。 这让我们减少了大量的样板代码，创建更可读的程序，提升应用的生产力。

In most of the code samples shown in this article streams were left unconsumed (we didn’t apply the close() method or a terminal operation). In a real app, don’t leave an instantiated streams unconsumed as that will lead to memory leaks.  

在大多数代码样例中，streams最后没有unconsumed，我们没有用close()方法或者最终操作。在实际的应用中，不要留一个实例化过的stream unconsumed,会造成内存泄漏。

The complete code samples that accompany the article are available at [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java).

 








