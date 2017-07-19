# Introduction to Java 8 Streams

From [java-8-streams-introduction](http://www.baeldung.com/java-8-streams-introduction)

## 1. Overview

In this article, we’ll have a quick look at one of the major pieces of new functionality Java 8 had added – Streams.

这篇文章，我们快速看下Java 8新加的几大主要部分之一，Streams.

We’ll explain what streams are about and showcase the creation and basic stream operations with simple examples.  

我们将会说明，streams是什么，并且用简单的例子列举stream的创建和基本操作。  

## 2. Stream API

One of the major new features in Java 8 is the introduction of the stream functionality – java.util.stream – which contains classes for processing sequences of elements.  

Java 8的几大主要新功能之一就是引入stream 功能，java.util.stream，这个包包含了用于处理序列的所有类。

The central API class is the Stream<T>. The following section will demonstrate how streams can be created using the existing data-provider sources.  

中心API类是Stream<T>。 下边的章节将会展示stream是怎样使用现有的数据源创建的。  

### 2.1. Stream Creation 

Streams can be created from different element sources e.g. collection or array with the help of stream() and of() methods:  

Streams 可以被从不同的元素创建。比如，collection或者array用stream()或者of()方法:  

```

	String[] arr = new String[]{"a", "b", "c"};
	Stream<String> stream = Arrays.stream(arr);
	stream = Stream.of("a", "b", "c");
``` 

A stream() default method is added to the Collection interface and allows creating a Stream<T> using any collection as an element source:  

stream()是加在Collection接口的默认的方法并且允许使用任何collection作为元素源创建Srtream<T>.

```
Stream<String> stream = list.stream();
``` 

### 2.2. Multi-threading with Streams  

Stream API also simplifies multithreading by providing the parallelStream() method that runs operations over stream’s elements in parallel mode.  

Stream API 还简化了多线程通过提供parallelStream()这个方法，这个方法以并行模式运行操作stream元素。

The code below allows to run method doWork() in parallel for every element of the stream:  

下边的代码以并行方式运行doWork()对每个元素： 

```
list.parallelStream().forEach(element -> doWork(element));
```
In the following section, we will introduce some of the basic Stream API operations.  

下边的章节，我们将介绍一些Stream API的基本操作。  

## 3. Stream Operations  

There are many useful operations that can be performed on a stream.

有很多有用的操作可以在stream上执行。  

They are divided into intermediate operations (return Stream<T>) and terminal operations (return a result of definite type). Intermediate operations allow chaining.  

这些操作被分成中间操作（仍然返回 Stream<T>）和最终操作（返回确切的类型）。 中间操作允许链式用法。  

It’s also worth noting that operations on streams don’t change the source.  

还需要值得提醒的是，stream上的操作，不改变源数据。  

Here’s a quick example: 

```
long count = list.stream().distinct().count();
```

So, the distinct() method represents an intermediate operation, which creates a new stream of unique elements of the previous stream. And the count() method is a terminal operation, which returns stream’s size.  

distinct()方法代表一个中间操作，这个方法创建了一个新的由之前所有元素的不同元素组成的stream。 这个count()方法是一个最终操作，返回stream的大小。  

### 3.1. Iterating

Stream API helps to substitute for, for-each and while loops. It allows concentrating on operation’s logic, but not on the iteration over the sequence of elements. For example:  

Stream API有了for-each和while循环的替代。 这允许了专注在操作逻辑本身，而不是在元素序列的迭代上。例如： 

```
for (String string : list) {
    if (string.contains("a")) {
        return true;
    }
}
```
This code can be changed just with one line of Java 8 code:  

Java 8 一行代码实现：

```
boolean isExist = list.stream().anyMatch(element -> element.contains("a"));
```

### 3.2. Filtering  

The filter() method allows us to pick stream of elements which satisfy a predicate.  

这个filter()方法允许我们挑选出一个新的stream,里边所有的元素都满足一个谓语 条件。

For example, consider the following list:  

例如，

```
ArrayList<String> list = new ArrayList<>();
list.add("One");
list.add("OneAndOnly");
list.add("Derek");
list.add("Change");
list.add("factory");
list.add("justBefore");
list.add("Italy");
list.add("Italy");
list.add("Thursday");
list.add("");
list.add("");
```

The following code creates a Stream<String> of the List<String>, finds all elements of this stream which contain char “d” and creates a new stream containing only the filtered elements:  

下边的代码用 List<String>创建了一个Stream<String>，找到这个stream中所有元素包含字符“d”的，并且创建一个新的stream只包含这些满足的元素：  

```
Stream<String> stream = list.stream().filter(element -> element.contains("d"));
```

### 3.3. Mapping  

To convert elements of a Stream by applying a special function to them and to collect these new elements into a Stream, we can use the map() method:  

为转化一个Stream的所有elements通过应用一个特别的方法并且收集这些新的elements到一个新的Stream,我们可以使用map()方法:  

```

	List<String> uris = new ArrayList<>();
	uris.add("C:\\My.txt");
	Stream<Path> stream = uris.stream().map(uri -> Paths.get(uri));
```

So, the code above converts Stream<String> to the Stream<Path> by applying a specific lambda expression to every element of the initial Stream.

上边的代码把Stream<String>转化成了Stream<Path>，通过对初始stream里的每个元素应用一个特定的lambda表达式。  

If you have a stream where every element contains its own sequence of elements and you want to create a stream of these inner elements, you should use the flatMap() method:  

如果有一个stream,它里边的element又包含一个自己的element序列，并且你想用里边的element创建一个新的stream，这时应该使用flatMap()方法：  

```

	List<Detail> details = new ArrayList<>();
	details.add(new Detail());
	Stream<String> stream
  	= details.stream().flatMap(detail -> detail.getParts().stream());
```

In this example, we have a list of elements of type Detail. The Detail class contains a field PARTS, which is a List<String>. With the help of the flatMap() method every element from field PARTS will be extracted and added to the new resulting stream. After that, the initial Stream<Detail> will be lost.  

在这个例子中，我们有一个Detail类型的list.Detail类包含一个List<String>类型的PARTS字段。用flatMap的时候，PARTS中的每个element都会被提取并且加入到一个新的stream中，最后，初始的Stream<Detail>将会丢失。  

### 3.4. Matching 匹配

Stream API gives a handy set of instruments to validate elements of a sequence according to some predicate. To do this one of the following methods can be used: anyMatch(), allMatch(), noneMatch(). Their names are self-explaining. Those are terminal operations which return a boolean.  

Stream API给了一个方便的工具来用一些谓语（条件）来验证一个序列的元素。为实现这个功能，以下的几个方法可以被使用，anyMatch(),allMatch(),noneMatch(). 这些方法的名字一看就懂。 这些都是最终操作，返回一个boolean类型。  

```

	boolean isValid = list.stream().anyMatch(element -> element.contains("h")); // true
	boolean isValidOne = list.stream().allMatch(element -> element.contains("h")); // false
	boolean isValidTwo = list.stream().noneMatch(element -> element.contains("h")); // false
```

### 3.5. Reduction  

Stream API allows reducing a sequence of elements to some value according to a specified function with the help of the reduce() method of the type Stream. This method takes two parameters: first – start value, second – an accumulator function.  

Stream API 允许转化elements序列成为一个与指定方法相关的值，用reduce()方法。 这个方法需要两个参数，第一个是起始值，第二个是一个累加方法。  

Imagine that you have a List<Integer> and you want to have a sum of all these elements and some initial Integer (in this example 23). So, you can run the following code and result will be 26 (23 + 1 + 1 + 1).  

想象有一个List<Integer>，并且你想要一个所有元素和一个起始值的和（在这个例子中是23）。所以，你可以运行以下的代码，结果会是26 (23 + 1 + 1 + 1)。  

```

	List<Integer> integers = Arrays.asList(1, 1, 1);
	Integer reduced = integers.stream().reduce(23, (a, b) -> a + b);
```

### 3.6. Collecting  

The reduction can also be provided by the collect() method of type Stream. This operation is very handy in case of converting a stream to a Collection or a Map and representing a stream in form of a single string. There is a utility class Collectors which provide a solution for almost all typical collecting operations. For some, not trivial tasks, a custom Collector can be created.  

这个reduction转化还可以被collect()方法提供。这个操作在把一个stream转化成Collection或者Map或者代表stream的一个形式的string，非常方便。 这里有一个方法类Collectors, 对几乎所有的典型的集合操作提供了解决。 对于一些重要的任务，可以自己创建一个自定义的Collector。  

```

	List<String> resultList 
	  = list.stream().map(element -> element.toUpperCase()).collect(Collectors.toList());
```

This code uses the terminal collect() operation to reduce a Stream<String> to the List<String>.  

这段代码使用了最终collect()操作，来把一个Stream<String>转化成List<String>.  

## Conclusions 总结  

In this article, we briefly touched upon Java streams – definitely one of the most interesting Java 8 features.  

在这篇文章中，我们简短的接触了Java stream。 绝对是Java 8最有意思的功能之一。  

There are much more advanced examples of using Streams; the goal of this write-up was only to provide a quick and practical introduction to what you can start doing with the functionality and as a starting point for exploring and further learning.  

这里还有更多的Stream的高级例子，这篇报道的目标是提供一个快速的实用的介绍对可以开始用这些功能做什么，并且作为一个探索和进一步学习的开始。  






