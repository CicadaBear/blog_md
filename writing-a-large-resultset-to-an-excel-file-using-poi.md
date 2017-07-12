# Writing a large resultset to an Excel file using POI

最近遇到一个问题，从数据库查询大量数据，写入到Excel中，耗时过长。 经理看Zipkin发现的有点数据耗时40多秒，并且经理发了一个相关技术链接，[streaming-mysql-results-using-java8-streams-and-spring-data]。 这个问题非常具有代表性，所以记录一下。 

本文主要的内容在翻译这篇技术博客[streaming-mysql-results-using-java8-streams-and-spring-data]。 

## Streaming MySQL Results Using Java 8 Streams and Spring Data JPA 

Since version 1.8, Spring data project includes an interesting feature - through a simple API call developer can request database query results to be returned as Java 8 stream. Where technically possible and supported by underlying database technology, results are being streamed one by one and are available for processing using stream operations. This technique can be particularly useful when processing large datasets (for example, exporting larger amounts of database data in a specific format) because, among other things, it can limit memory consumption in the processing layer of the application. In this article I will discuss some of the benefits (and gotchas!) when Spring Data streaming is used with MySQL database.  

从Spring Data 1.8开始，Spring Data项目包含了一个有趣的功能，通过一个简单的API调用，开发者可以请求查询结果返回一个Java 8的stream。 在这个地方，技术的可能性和被底层数据库技术支持，结果被以流的形式一个接一个的传回来，并且可以使用stream操作。 这个技术可能特别有用当处理大量数据的时候。 （比如，以一个特定的格式导出大量数据。）因为，除了其他的原因之外，它可以限制内存的消耗在应用的处理层。 这篇文章中，我们将会探讨使用Spring Data streaming结合MySQL数据库使用的好处和陷阱。  

Naive approaches to fetching and processing a larger amount of data (by larger, I mean datasets that do not fit into the memory of the running application) from the database will often result with running out of memory. This is especially true when using ORMs / abstraction layers such as JPA where you don't have access to lower level facilities that would allow you to manually manage how data is fetched from the database. Typically, at least with the stack that I'm usually using - MySQL, Hibernate/JPA and Spring Data - the whole resultset of a large query will be fetched entirely either by MySQL's JDBC driver or one of the aforementioned frameworks that come after it. This will lead to OutOfMemory exceptions if the resultset is sufficiently large.   

原生方法获取和处理大量数据从数据库获取，会经常内存溢出（大量数据指的是，数据不能适应运行的应用的内存）。 这个问题在ORMS上或者抽象层比如JPA,在这里，我们没有访问底层的设施。 有代表性的，至少在这种情况下，我使用MySQL，Hibernate/JPA还有Spring Data。 这个大查询的查询结果将会被MySQL的JDBC驱动或者上述的框架。 这会造成内存溢出异常，如果返回结果足够大。  


### Solution Using Paging  使用分页解决

Let's focus on a single example - exporting results of a large query as a CSV file. When Let's focus on a single example - exporting results of a large query as a CSV file. When presented with this problem and when I want to stay in Spring Data/JPA world, I usually settle for a paging solution. The query is broken down to smaller queries that each return one page of results, each with a limited size. Spring Data offers nice paging/slicing feature that makes this approach easy to implement. Spring Data's PageRequests get translated to limit/offset queries in MySQL. There are caveats though. When using JPA, entities get cached in EntityManager's cache. This cache needs to be cleared to enable garbage collector to remove the old result objects from the memory. with this problem and when I want to stay in Spring Data/JPA world, I usually settle for a paging solution. The query is broken down to smaller queries that each return one page of results, each with a limited size. Spring Data offers nice paging/slicing feature that makes this approach easy to implement. Spring Data's PageRequests get translated to limit/offset queries in MySQL. There are caveats though. When using JPA, entities get cached in EntityManager's cache. This cache needs to be cleared to enable garbage collector to remove the old result objects from the memory.  

我们关注一个单一的例子。 导出大量查询结果到一个CSV文件中。 当出现了这个问题，并且我还想继续使用Spring Data/JPA, 我通常用分页解决。 这个大的查询被拆分成小的查询，每个小查询返回一页的结果，每一页只有有限的数据。 Spring Data的分页请求被翻译成limit/offset查询在MySQL中。 不过这里还有警告。 当使用JPA的时候，实体都会被缓存到EntityManager的缓存中。 这个缓存需要被清理来使GC垃圾回收从内存移除掉旧的对象。  

Let's take look how an actual implementation of paging strategy behaves in practice. For testing purposes I will use a small [application based on Spring Boot, Spring Data, Hibernate/JPA and MySQL](https://github.com/knes1/todo/tree/mysql-streaming-test). It's a todo list management webapp and it has a feature to download all todos as a CSV file. Todos are stored in a single MySQL table. The table has been filled with 1 million entries. Here's the code for the paging/slicing export function:

我们看一下一个实际的分页策略的实现如何表现。 为了测试的目的，我将创建一个小应用，基于Spring Boot, Spring Data, Hibernate/JPA还有MySQL，[github]((https://github.com/knes1/todo/tree/mysql-streaming-test). 这是个待处理事务清单管理的webapp，有一个功能是下载所有的将要做的事务作为一个CSV文件。 事务列表保存在一个MySQL表中。 这个表有一百万条数据。 下边是分页/切片导出的方法:

```
@RequestMapping(value = "/todos2.csv", method = RequestMethod.GET)
public void exportTodosCSVSlicing(HttpServletResponse response) {
	final int PAGE_SIZE = 1000;
	response.addHeader("Content-Type", "application/csv");
	response.addHeader("Content-Disposition", "attachment; filename=todos.csv");
	response.setCharacterEncoding("UTF-8");
	try {
		PrintWriter out = response.getWriter();
		int page = 0;
		Slice<Todo> todoPage;
		do {
			todoPage = todoRepository.findAllBy(new PageRequest(page, PAGE_SIZE));
			for (Todo todo : todoPage) {
				String line = todoToCSV(todo);
				out.write(line);
				out.write("\n");
			}
			entityManager.clear();
			page++;
		} while (todoPage.hasNext());
		out.flush();
	} catch (IOException e) {
		log.info("Exception occurred " + e.getMessage(), e);
		throw new RuntimeException("Exception occurred while exporting results", e);
	}
}
```
This how memory usage looks like while export operation is in progress: 
下边是这个导出操作过程中的内存使用情况：
![mem-paging-slicing.png](http://ogyd2yldv.bkt.clouddn.com/mem-paging-slicing.png)

Memory usage graph has a shape of a saw-tooth: memory usage grows as entries are fetched from the database until GC kicks in and cleans up the entries that have already been outputted and cleared from EntityManager's cache. Paging approach works great but it definitely has room from improvement:  

内存使用图表有个锯齿形状：内存使用随着从数据库获取到实体而增长，直到GC开始生效，清除了已经输出的，并且已经被EntityManager缓存清除的实体。 分页方法工作的非常好，但是肯定还有提升的空间。  

* We're issuing 1000 database queries (number of entries / PAGE_SIZE) to complete the export. It would be better if we could avoid the overhead of executing those queries. 我们触发1000次数据库查询（总条数除以每页条数），来完成导出。 如果能避免过高的查询次数，将会更好。  

* Did you notice how the raising slope of the tooths on the graph is less and less steep as the export progresses and the distance between the peaks increases? It seems that the rate at which new entires are fetched from DB gets slower and slower. The reason for this is MySQL's limit/offset performance characteristic - as offset gets larger it takes more and more time to find and return the selected rows. 不知道你注意到没有，图表上锯齿的上升倾斜越来越不那么陡峭了并且顶点之间的距离越来越大了。 这似乎是，新的实体从数据库中获取出的速度越来越慢。 这个原因是MySQL的limit/offset性能特点，offset越大，就会用更多的时间返回数据。  


Can we improve the above using new streaming functionality available in Spring Data 1.8? Let's try. 

我们能不能使用Spring Data1.8的新功能streaming来提高上边的方法，试一下。  

## Streaming Functionality in Spring Data 1.8

Spring Data 1.8 introduced support for streaming resultsets. Repositories can now declare methods that return Java 8 streams of entity objects. For example, it's now possible to add a method with the following signature to a repository: 

Spring Data 1.8 引入了对streaming resultset的支持。 Repositories现在可以声明一个方法返回Java 8 实体的Stream类型。 比如，现在加一个如下的方法：  

```
@Query("select t from Todo t")
Stream<Todo> streamAll();
```

Spring Data will use techniques specific to a particular JPA implementation (e.g. Hibernate's, EclipseLink's etc.) to stream the resultset. Let's re-implement the CSV exporting using this streaming capability:  

Spring Data 将会使用特定的技术对于特定的JPA实现（比如，Hibernate，EclipseLink)来stream resultset。 我们重新实现一下使用streaming导出CSV：  

```
@RequestMapping(value = "/todos.csv", method = RequestMethod.GET)
@Transactional(readOnly = true)
public void exportTodosCSV(HttpServletResponse response) {
	response.addHeader("Content-Type", "application/csv");
	response.addHeader("Content-Disposition", "attachment; filename=todos.csv");
	response.setCharacterEncoding("UTF-8");
	try(Stream<Todo> todoStream = todoRepository.streamAll()) {
		PrintWriter out = response.getWriter();
		todoStream.forEach(rethrowConsumer(todo -> {
			String line = todoToCSV(todo);
			out.write(line);
			out.write("\n");
			entityManager.detach(todo);
		}));
		out.flush();
	} catch (IOException e) {
		log.info("Exception occurred " + e.getMessage(), e);
		throw new RuntimeException("Exception occurred while exporting results", e);
	}
}
```  
I started the export as usual, however the results are not showing up. What's happening?  

我像往常一样开始了导出，但是结果却没有出现，发生了什么？  

![mem-streaming-outofmem.png](http://ogyd2yldv.bkt.clouddn.com/mem-streaming-outofmem.png)  

It seems that we have run out of memory. Additionally, no results were written to HttpServletResponse. Why isn't this working? After digging into source code of org.springframework.data.jpa.provider.PersistenceProvider one can find out that Spring Data is using scrollable resultsets to implement resultset streams. Googling about scrollable resultsets and MySQL shows that there are gotchas when using them. For instance, here's a quote from [MySQL's JDBC driver's documentation](http://dev.mysql.com/doc/connector-j/en/connector-j-reference-implementation-notes.html):  

看起来好像，内存溢出了。此外，没有结果被写出到HttpServletResponse。 为什么这个不工作？在看了PersistenceProvider的源码之后发现，Spring Data是在使用scrollable resultsets来实现resultset streams。 谷歌搜索scrollable resultsets and MySQL，发现使用它们的时候有个坑，例如，这里有一个MySQL JDBC驱动的文档的引用。  

>    By default, ResultSets are completely retrieved and stored in memory. In most cases this is the most efficient way to operate and, due to the design of the MySQL network protocol, is easier to implement. If you are working with ResultSets that have a large number of rows or large values and cannot allocate heap space in your JVM for the memory required, you can tell the driver to stream the results back one row at a time. To enable this functionality, create a Statement instance in the following manner:   
>    
>    默认情况下，所有的ResultSets重新获取并且存储在内存中。 在大多数案例中，这是最有效率的方法，并且，由于MySQL的网络协议，很容易实现。 如果涉及到的ResultSets有大量的行或者大值，你可以告诉驱动来一次stream回（返回）一行。 要开启这个功能，用下边的方法创建一个statement实例：

>    stmt = conn.createStatement(java.sql.ResultSet.TYPE_FORWARD_ONLY,java.sql.ResultSet.CONCUR_READ_ONLY);
>    stmt.setFetchSize(Integer.MIN_VALUE);

>    The combination of a forward-only, read-only result set, with a fetch size of Integer.MIN_VALUE serves as a signal to the driver to stream result sets row-by-row. After this, any result sets created with the statement will be retrieved row-by-row.  
>    
>    这个forward-only,read-only还有fetch-size的值Integer.MIN_VALUE，的组合作为一个对驱动的信号来stream 结果数据一行接一行。 在这之后，任何的用这个statement产生的result set都将会被一行一行的获取。  

>    There are some caveats with this approach. You must read all of the rows in the result set (or close it) before you can issue any other queries on the connection, or an exception will be thrown.  

>    对于这个方法还有一些警告。 必须读取所有的行在这个result set中，或者关闭它，在你在这个连接上可能触发任何其他查询之前，否则，会抛出异常。  

Ok, it seems that when using MySQL in order to really stream the results we need to satisfy three conditions:  

要想使用MySQL来stream results,我们需要满足三个条件： 

* Forward-only resultset
* Read-only statement
* Fetch-size set to Integer.MIN_VALUE

Forward-only seems to be set already by Spring Data so we don't have to do anything special about that. Our code sample already has @Transactional(readOnly = true) annotation which is enough to satisfy the second criteria. What seems to be missing is the fetch-size. We can set it up using query hints on the repository method:  

Forward-only好像已经被Spring Data设置上了，所以，我们不需要做任何相关的配置。 我们的代码样例已经有了@Transactional(readOnly = true)注解，这个注解就可以满足第二个条件了。好像fetch-size丢失了，我们可以使用query hint在repository的方法上:

```
...
import static org.hibernate.jpa.QueryHints.HINT_FETCH_SIZE;

@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {

	@QueryHints(value = @QueryHint(name = HINT_FETCH_SIZE, value = "" + Integer.MIN_VALUE))
	@Query(value = "select t from Todo t")
	Stream<Todo> streamAll();
	
	...
}
```
With query hints in place, let's run export again:

![mem-streaming.png](http://ogyd2yldv.bkt.clouddn.com/mem-streaming.png)

Everything is working now, and it seems it's much more efficient than the paging approach:

所有的都工作了，看起来跟分页的方法比较更有效率了。  

* When streaming, export is finished in about 9 seconds vs about 137 seconds when using paging 当streaming的时候，导出在9秒后结束，当使用分页方法的时候，耗时137秒

* It seems offset performance, query overhead and result preloading can really hurt paging approach when dataset is sufficiently large 看起来offset性能，过多查询，结果预加载会真的伤害分页方法，当数据量非常大的时候。  

## Conclusions 总结

* We've seen significant performance improvements when using streaming (via scrollable resultsets) vs paging, admittedly in a very specific task of exporting data. 我们已经看到了显著的增长，当使用streaming(通过 scrollable resultsets)对比分页，公认的在非常特殊的额导出数据时使用。

* Spring Data's new features give really convenient access to scrollable resultsets via streams. Spring Data的新功能非常方便的访问scrollable resultsets通过streams.

* There are gotchas to get it working with MySQL, but they are manageable. 这里跟MySQL工作有坑，但是可以管理。  

* There are further restrictions when reading scrollable result sets in MySQL - no statement may be issued through the same database connection until the resultset is fully read. 这里还有更多的限制, 当在MySQL读scrollable result sets时，当读完之前不能有任何其他查询操作。  

* The export works fine because we are writing results directly to HttpServletResponse. If we were using default Spring's message converters (e.g. returning stream from the controller method ) then there's a good chance this would not work as expected. Here's an interesting [article](https://www.airpair.com/java/posts/spring-streams-memory-efficiency) on this subject. 这个导出工作顺利是因为，我们直接把结果写入到HttpServletResponse中。 如果，我们使用默认的Spring's message converters（从controller方法中返回stream), 然后这里有个好机会，不会按照预期工作。 这里有一个有趣的文章。  

I would love to try the tests with other database and explore possibilities of streaming results via Spring message converters as hinted in article linked above. If you'd like to experiment yourself, the test application is available on github. I hope you found the article interesting and I welcome your comments in the comment section below.

文章主要部分完了。

Apache POI库 XSSFWorkbook全部Excel（-2007）文档全部加载到内存， SXSSFWorkbook Excel(2007-)可以只把固定行加入到内存。 100行之外的部分以文件保存。 最后写入到输出流。  

[Apache POI 文档](https://poi.apache.org/spreadsheet/index.html)  

HSSF is the POI Project's pure Java implementation of the Excel '97(-2007) file format. XSSF is the POI Project's pure Java implementation of the Excel 2007 OOXML (.xlsx) file format.  

HSSF是POI项目的纯Java对于Excel '97(-2007) .xls文件的实现。 XSSF是Excel 2007 .xlsx文件格式的Java实现。

HSSF and XSSF provides ways to read spreadsheets create, modify, read and write XLS spreadsheets. They provide:  

HSSF and XSSF提供了电子表格XLS的增改读写的方法。 具体是：  

* low level structures for those with special needs 
* an eventmodel api for efficient read-only access
* a full usermodel api for creating, reading and modifying XLS files

SXSSF (package: org.apache.poi.xssf.streaming) is an API-compatible streaming extension of XSSF to be used when very large spreadsheets have to be produced, and heap space is limited. SXSSF achieves its low memory footprint by limiting access to the rows that are within a sliding window, while XSSF gives access to all rows in the document. Older rows that are no longer in the window become inaccessible, as they are written to the disk.  

SXSSF（位于包：org.apache.poi.xssf.streaming）是一个API兼容的XSSF的streaming拓展，被用于特大的电子表格，而且当堆内存有限的时候。 SXSSF通过限制访问行数实现了小内存消耗，在一个滑动窗口，而XSSF提供的是访问文档中的所有行。 之前的行，不在窗口中的，变得不可访问，它们被写入到了文件中。

You can specify the window size at workbook construction time via new SXSSFWorkbook(int windowSize) or you can set it per-sheet via SXSSFSheet#setRandomAccessWindowSize(int windowSize)  

可以在workbook工作簿创建时指定窗口的大小，通过new SXSSFWorkbook(窗口大小/行数)，或者也可以在每一个sheet上设置 SXSSFSheet#setRandomAccessWindowSize(int windowSize)。  

When a new row is created via createRow() and the total number of unflushed records would exceed the specified window size, then the row with the lowest index value is flushed and cannot be accessed via getRow() anymore.  

当一个新的行通过createRow()被创建，并且没有写入文件中的记录将要超过指定的窗口大小，然后索引最小的行将会被写入文件，并且再也不能通过getRow访问。  

The default window size is 100 and defined by SXSSFWorkbook.DEFAULT_WINDOW_SIZE.  

默认的窗口大小是100，而且是被SXSSFWorkbook.DEFAULT_WINDOW_SIZE定义的。  

A windowSize of -1 indicates unlimited access. In this case all records that have not been flushed by a call to flushRows() are available for random access.  

当窗口大小为-1，表示无限的访问。 在这个情况下，所有的记录，没有被flushRows()写入到文件的，都可以被随机访问。  

Note that SXSSF allocates temporary files that you must always clean up explicitly, by calling the dispose method.  

注意，SXSSF分配了临时文件，这些临时文件必须显式清理，通过调用dispose方法。  

SXSSFWorkbook defaults to using inline strings instead of a shared strings table. This is very efficient, since no document content needs to be kept in memory, but is also known to produce documents that are incompatible with some clients. With shared strings enabled all unique strings in the document has to be kept in memory. Depending on your document content this could use a lot more resources than with shared strings disabled.  

SXSSFWorkbook默认使用inline strings 而不是 shared strings table 共享字符串表。 这样会更有效率，既然没有文档内容需要保持在内存中，但是也会跟一些客户端不兼容。 开启shared strings共享字符串，所有的唯一的文档中的字符串必须保持在内存中。 取决于你的文档内容，开启共享字符串可能会使用更多的资源比关闭它。  

Please note that there are still things that still may consume a large amount of memory based on which features you are using, e.g. merged regions, hyperlinks, comments, ... are still only stored in memory and thus may require a lot of memory if used extensively.  

请注意，这里还是有东西会消耗大量的内存，要看你使用的功能。  

Carefully review your memory budget and compatibility needs before deciding whether to enable shared strings or not.

The example below writes a sheet with a window of 100 rows. When the row count reaches 101, the row with rownum=0 is flushed to disk and removed from memory, when rownum reaches 102 then the row with rownum=1 is flushed, etc.

。。。

上边的文章说到了Spring Data中如何使用repository方法一行一行获取大量数据，下边的例子是如何使用entityManager实现。  

```
 
		Session hibernateSession = entityManager.unwrap(Session.class);
        Session session = hibernateSession.getSessionFactory().openSession();

        ScrollableResults scResults = session.createQuery(sql + querySql)
                .setCacheable(false)
                .setFetchSize(Integer.MIN_VALUE)
                .setReadOnly(true)
                .scroll(ScrollMode.FORWARD_ONLY);
        List<List<String>> dataList = new ArrayList<>();
        while (scResults.next()) {
		}
		session.close();
```

*** 
### Reference


[streaming-mysql-results-using-java8-streams-and-spring-data]

[out-of-memory-error-java-heap-space-while-writing-to-excel](https://stackoverflow.com/questions/14259568/out-of-memory-error-java-heap-space-while-writing-to-excel)  

[SXSSFWorkbook](http://poi.apache.org/apidocs/org/apache/poi/xssf/streaming/SXSSFWorkbook.html)  

[how-to-sxssf](https://poi.apache.org/spreadsheet/how-to.html#sxssf)

[hibernate-scrollableresults-and-scrollmode-example](http://www.concretepage.com/hibernate/hibernate-scrollableresults-and-scrollmode-example)

[get-hibernate-sessionfactory-from-jpas-entitymanagerfactory](https://stackoverflow.com/questions/19030022/get-hibernate-sessionfactory-from-jpas-entitymanagerfactory)



[streaming-mysql-results-using-java8-streams-and-spring-data]:http://knes1.github.io/blog/2015/2015-10-19-streaming-mysql-results-using-java8-streams-and-spring-data.html