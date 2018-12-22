# Apache CXF Support for RESTful Web Services 

[original](https://www.baeldung.com/apache-cxf-rest-api)

原来Web Service还可以支持Restful, 其实我没有必要累半天，写半天，对我有用的就是知道Web Service还可以这样玩，仅此而已。还有一些测试用例的Tips&Tricks.

## 1. Overview 概览

> This tutorial introduces Apache CXF as a framework compliant with the JAX-RS standard, which defines support of the Java ecosystem for the REpresentational State Transfer (REST) architectural pattern.  
> 
> Specifically, it describes step by step how to construct and publish a RESTful web service, and how to write unit tests to verify a service.
>
> This is the third in a series on Apache CXF; the first one focuses on the usage of CXF as a JAX-WS fully compliant implementation. The second article provides a guide on how to use CXF with Spring.

这篇教程介绍了，Apache CXF作为一个遵从JAX-RS标准的框架，定义了Java生态对于REST架构模式的支持。  

具体来说，这篇教程一步一步描述了怎样创建并且发布一个Restful的web service, 而且还有怎样写单元测试来验证这个服务。  

这是Apache CXF系列的第三篇文章，第一篇关注的是CXF作为一个JAX-WS标准的完全遵循实现。第二篇文章提供了一个怎样和Spring一起使用CXF的指南。 

## 2. Maven Dependencies 

> The first required dependency is org.apache.cxf:cxf-rt-frontend-jaxrs. This artifact provides JAX-RS APIs as well as a CXF implementation: 

首先必须的依赖是org.apache.cxf:cxf-rt-frontend-jaxrs，这个库提供了JAX-RS APIs还有一个CXF的实现： 

	<dependency>
    	<groupId>org.apache.cxf</groupId>
    	<artifactId>cxf-rt-frontend-jaxrs</artifactId>
    	<version>3.1.7</version>
	</dependency>

> In this tutorial, we use CXF to create a Server endpoint to publish a web service instead of using a servlet container. Therefore, the following dependency needs to be included in the Maven POM file: 

在这篇文章中，我们使用CXF来创建一个Server endpoint（指jetty）来发布一个web service，而不是使用一个servlet container（指tomcat）. 因此，以下的以来需要被引入到Maven POM文件:  

	<dependency>
    	<groupId>org.apache.cxf</groupId>
    	<artifactId>cxf-rt-transports-http-jetty</artifactId>
    	<version>3.1.7</version>
	</dependency>

> Finally, let’s add the HttpClient library to facilitate unit tests: 

最后我们加上HttpClient库莱简化单元测试: 

	<dependency>
    	<groupId>org.apache.httpcomponents</groupId>
    	<artifactId>httpclient</artifactId>
    	<version>4.5.2</version>
	</dependency>

> [Here](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-frontend-jaxrs%22) you can find the latest version of the cxf-rt-frontend-jaxrs dependency. You may also want to refer to [this link](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-transports-http-jetty%22) for the latest versions of the org.apache.cxf:cxf-rt-transports-http-jetty artifacts. Finally, the latest version of httpclient can be found [here](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.httpcomponents%22%20AND%20a%3A%22httpclient%22).  

这里你可以找到cxf-rt-frontend-jaxrs的最新版本。。。

## 3. Resource Classes and Request Mapping  

> Let’s start implementing a simple example; we’re going to set up our REST API with two resources Course and Student.
> 
> We’ll start simple and move towards a more complex example as we go.  

我们先来实现一个简单的例子，我们将要设置我们的REST API实现两个资源 Course and Student. 

我们将会开始于简单，然后随着我们的进行，向前发展成为一个更加复杂的例子。

## 3.1. The Resources  

> Here is the definition of the Student resource class: 

``` Java 
@XmlRootElement(name = "Student")
public class Student {
    private int id;
    private String name;
 
    // standard getters and setters
    // standard equals and hashCode implementations
 
}
``` 

> Notice we’re using the @XmlRootElement annotation to tell JAXB that instances of this class should be marshaled to XML. 

注意我们使用@XmlRootElement注解来告诉JAXB，这个类的实例应该被排列成XML. 

> Next, comes the definition of the Course resource class:  

```
@XmlRootElement(name = "Course")
public class Course {
    private int id;
    private String name;
    private List<Student> students = new ArrayList<>();
 
    private Student findById(int id) {
        for (Student student : students) {
            if (student.getId() == id) {
                return student;
            }
        }
        return null;
    }

``` 

```
// standard getters and setters
// standard equals and hasCode implementations
``` 

> Finally, let’s implement the CourseRepository – which is the root resource and serves as the entry point to web service resources:  

最后，我们实现CourseRepository - 这个是根资源，并且作为一个web service的资源入口来服务: 

```
@Path("course")
@Produces("text/xml")
public class CourseRepository {
    private Map<Integer, Course> courses = new HashMap<>();
 
    // request handling methods
 
    private Course findById(int id) {
        for (Map.Entry<Integer, Course> course : courses.entrySet()) {
            if (course.getKey() == id) {
                return course.getValue();
            }
        }
        return null;
    }
}
```  

> Notice the mapping with the @Path annotation. The CourseRepository is the root resource here, so it’s mapped to handle all URLS starting with course.
>
> The value of @Produces annotation is used to tell the server to convert objects returned from methods within this class to XML documents before sending them to clients. We’re using JAXB here as the default since no other binding mechanisms are specified.

注意@Path注解的映射。 这个CourseRepository 是根资源，所以，它映射来处理所有以course开头的url。 

@Produces注解里的值告诉服务器来转化这个类的方法返回的对象为XML文档，在把它们返回到客户端之前。我们在这里使用JAXB作为默认，既然没有其他的绑定机制是指定的。 


## 3.2. Simple Data Setup 简单的数据设置 

> Because this is a simple example implementation, we’re using in-memory data instead of a full-fledged persistent solution.
>
> With that in mind, let’s implement some simple setup logic to populate some data into the system: 

因为这是个简单的实现例子，我们使用内存中的数据，而不是完全的持久化解决方案。 

有这个在脑子里，我们实现一些简单的设置逻辑来向系统中添加一些数据:  

> Methods within this class that take care of HTTP requests are covered in the next subsection. 

这个类里边考虑包含Http requests的方法，在下一个子章节中。 

## 3.3. The API – Request Mapping Methods  

> Now, let’s go to the implementation of the actual REST API.
>
> We’re going to start adding API operations – using the @Path annotation – right in the resource POJOs.
>
> It’s important to understand that is a significant difference from the approach in a typical Spring project – where the API operations would be defined in a controller, not on the POJO itself.
>
> Let’s start with mapping methods defined inside the Course class: 

现在，我们来实现实际的REST API. 

我们将要开始添加API操作，使用@Path注解，就在POJO资源中。

理解跟典型的Spring项目相比非常大的不同是，API操作定义在controller中，而不是POJO自己中非常重要。

我们以在Course类中定义映射方法为开始: 

```
@GET
@Path("{studentId}")
public Student getStudent(@PathParam("studentId")int studentId) {
    return findById(studentId);
}
``` 

> Simply put, the method is invoked when handling GET requests, denoted by the @GET annotation.
>
> Noticed the simple syntax of mapping the studentId path parameter from the HTTP request.
>
> We’re then simply using the findById helper method to return the corresponding Student instance.
>
> The following method handles POST requests, indicated by the @POST annotation, by adding the received Student object to the students list: 

简言之, 这个被@GET注解的方法当处理GET请求的时候会被调用。

注意，简单语法映射了the studentId path parameter from the HTTP request.

我们然后使用findById helper方法来返回对应的Student实体。 

以下用@POST标识的方法处理POST请求，通过把请求添加接收到的Student对象到students列表中。

```
@POST
@Path("")
public Response createStudent(Student student) {
    for (Student element : students) {
        if (element.getId() == student.getId() {
            return Response.status(Response.Status.CONFLICT).build();
        }
    }
    students.add(student);
    return Response.ok(student).build();
}
```

> This returns a 200 OK response if the create operation was successful, or 409 Conflict if an object with the submitted id is already existent.
>
> Also note that we can skip the @Path annotation since its value is an empty String.
>
> The last method takes care of DELETE requests. It removes an element from the students list whose id is the received path parameter and returns a response with OK (200) status. In case there are no elements associated with the specified id, which implies there is nothing to be removed, this method returns a response with Not Found (404) status: 

如果创建操作成功，这个返回200 OK 响应，或者如果提交的对象的id已经存在了，就返回409冲突。

还有注意，我们可以略过@Path注解，既然它的值是空字符串。

最后一个方法，负责处理DELETE请求。它从students列表中移除一个id和path变量中相同的元素并且返回一个200响应。万一列表里没有制定的Id的元素，这时候意味着没有东西将被移除，这个方法返回404响应。 

```
@DELETE
@Path("{studentId}")
public Response deleteStudent(@PathParam("studentId") int studentId) {
    Student student = findById(studentId);
    if (student == null) {
        return Response.status(Response.Status.NOT_FOUND).build();
    }
    students.remove(student);
    return Response.ok().build();
}
```

> Let’s move on to request mapping methods of the CourseRepository class.
>
> The following getCourse method returns a Course object that is the value of an entry in the courses map whose key is the received courseId path parameter of a GET request. Internally, the method dispatches path parameters to the findById helper method to do its job. 

我们前进到CourseRepository类的请求映射方法。  

以下getCourse方法返回一个Course对象，这个Course对象是Courses Map中，key是接收到的path参数。内部地，这个方法派发了patt参数到findById hepler方法中，来完成这个工作。 

```
@GET
@Path("courses/{courseId}")
public Course getCourse(@PathParam("courseId") int courseId) {
    return findById(courseId);
}
``` 

> The following method updates an existing entry of the courses map, where the body of the received PUT request is the entry value and the courseId parameter is the associated key: 

下边的方法更新了一个已经存在的courses Map entry，PUT请求的body体是entry的值，courseId参数是entry的key: 

```
@PUT
@Path("courses/{courseId}")
public Response updateCourse(@PathParam("courseId") int courseId, Course course) {
    Course existingCourse = findById(courseId);        
    if (existingCourse == null) {
        return Response.status(Response.Status.NOT_FOUND).build();
    }
    if (existingCourse.equals(course)) {
        return Response.notModified().build();    
    }
    courses.put(courseId, course);
    return Response.ok().build();
}
```

> This updateCourse method returns a response with OK (200) status if the update is successful, does not change anything and returns a Not Modified (304) response if the existing and uploaded objects have the same field values. In case a Course instance with the given id is not found in the courses map, the method returns a response with Not Found (404) status.
>
> The third method of this root resource class does not directly handle any HTTP request. Instead, it delegates requests to the Course class where requests are handled by matching methods:  

根资源下的第三个方法不直接处理HTTP请求。而是，为Course类代理请求，Course类中的对应方法来处理请求。

```
@Path("courses/{courseId}/students")
public Course pathToStudent(@PathParam("courseId") int courseId) {
    return findById(courseId);
}
``` 

> We have shown methods within the Course class that process delegated requests right before.  

我们已经展示过Course类的处理被代理的请求的方法了，就在前文。注意这个接口是根据couseraId查找students列表。


## 4. Server Endpoint 

> This section focuses on the construction of a CXF server, which is used for publishing the RESTful web service whose resources are depicted in the preceding section. The first step is to instantiate a JAXRSServerFactoryBean object and set the root resource class:  

这一章关注于构建CXF server，被用来发布RESTful web service，web service的资源已经在前边描述了。第一步是实例化一个JAXRSServerFactoryBean对象，并且设置根资源类: 

```
JAXRSServerFactoryBean factoryBean = new JAXRSServerFactoryBean();
factoryBean.setResourceClasses(CourseRepository.class);
```

> A resource provider then needs to be set on the factory bean to manage the life cycle of the root resource class. We use the default singleton resource provider that returns the same resource instance to every request:  

一个资源提供者，然后需要被设置在工厂bean上类管理根资源类的生命周期。我们使用默认的单例资源提供者来返回相同的资源实例为每个请求: 

```
factoryBean.setResourceProvider(
  new SingletonResourceProvider(new CourseRepository()));
```

> We also set an address to indicate the URL where the web service is published: 

```
factoryBean.setAddress("http://localhost:8080/");
``` 

> Now the factoryBean can be used to create a new server that will start listening for incoming connections: 

```
Server server = factoryBean.create();
```

> All the code above in this section should be wrapped in the main method: 

```
public class RestfulServer {
    public static void main(String args[]) throws Exception {
        // code snippets shown above
    }
}
``` 

> The invocation of this main method is presented in section 6 



## 5. Test Cases 测试用例


> This section describes test cases used to validate the web service we created before. Those tests validate resource states of the service after responding to HTTP requests of the four most commonly used methods, namely GET, POST, PUT, and DELETE. 

这个章节描述了测试用例用来验证我们之前创建的web service. 这些测试，验证资源服务的响应状态。

### 5.1. Preparation 

> First, two static fields are declared within the test class, named RestfulTest:  

首先两个静态变量。

```
private static String BASE_URL = "http://localhost:8080/baeldung/courses/";
private static CloseableHttpClient client;
```

> Before running tests we create a client object, which is used to communicate with the server and destroy it afterward: 

在运行测试之前，我们创建一个client对象，这个对象被用来跟server通信，并且运行之后销毁它:  

```
@BeforeClass
public static void createClient() {
    client = HttpClients.createDefault();
}
     
@AfterClass
public static void closeClient() throws IOException {
    client.close();
}
``` 

> The client instance is now ready to be used by test cases. client准备好了

### 5.2. GET Requests 

In the test class, we define two methods to send GET requests to the server running the web service.

The first method is to get a Course instance given its id in the resource: 

```
private Course getCourse(int courseOrder) throws IOException {
    URL url = new URL(BASE_URL + courseOrder);
    InputStream input = url.openStream();
    Course course
      = JAXB.unmarshal(new InputStreamReader(input), Course.class);
    return course;
}
``` 

The second is to get a Student instance given the ids of the course and student in the resource:  

```
private Student getStudent(int courseOrder, int studentOrder)
  throws IOException {
    URL url = new URL(BASE_URL + courseOrder + "/students/" + studentOrder);
    InputStream input = url.openStream();
    Student student
      = JAXB.unmarshal(new InputStreamReader(input), Student.class);
    return student;
}
``` 

These methods send HTTP GET requests to the service resource, then unmarshal XML responses to instances of the corresponding classes. Both are used to verify service resource states after executing POST, PUT, and DELETE requests.  

### 5.3. POST Requests  

This subsection features(特写了) two test cases for POST requests, illustrating operations of the web service when the uploaded Student instance leads to a conflict and when it is successfully created.  

In the first test, we use a Student object unmarshaled（反序列） from the conflict_student.xml file, located on the classpath with the following content:

```
<Student>
    <id>2</id>
    <name>Student B</name>
</Student>
``` 

This is how that content is converted to a POST request body:  

```
HttpPost httpPost = new HttpPost(BASE_URL + "1/students");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("conflict_student.xml");
httpPost.setEntity(new InputStreamEntity(resourceStream));
``` 

The Content-Type header is set to tell the server that the content type of the request is XML: 

```
httpPost.setHeader("Content-Type", "text/xml");
```

Since the uploaded Student object is already existent in the first Course instance, we expect that the creation fails and a response with Conflict (409) status is returned. The following code snippet verifies the expectation: 

```
HttpResponse response = client.execute(httpPost);
assertEquals(409, response.getStatusLine().getStatusCode());
```

In the next test, we extract the body of an HTTP request from a file named created_student.xml, also on the classpath. Here is content of the file:

```
<Student>
    <id>3</id>
    <name>Student C</name>
</Student>
```

Similar to the previous test case, we build and execute a request, then verify that a new instance is successfully created:  

```
HttpPost httpPost = new HttpPost(BASE_URL + "2/students");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("created_student.xml");
httpPost.setEntity(new InputStreamEntity(resourceStream));
httpPost.setHeader("Content-Type", "text/xml");
         
HttpResponse response = client.execute(httpPost);
assertEquals(200, response.getStatusLine().getStatusCode());
```

We may confirm new states of the web service resource: 

```
Student student = getStudent(2, 3);
assertEquals(3, student.getId());
assertEquals("Student C", student.getName());
```

This is what the XML response to a request for the new Student object looks like:  

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Student>
    <id>3</id>
    <name>Student C</name>
</Student>
```

### 5.4. PUT Requests 

Let’s start with an invalid update request, where the Course object being updated does not exist. Here is content of the instance used to replace a non-existent Course object in the web service resource: 

```
<Course>
    <id>3</id>
    <name>Apache CXF Support for RESTful</name>
</Course>
```
.
.
.

### 5.5. DELETE Requests  

.
.
.

## 6. Test Execution 测试执行

> Section 4 described how to create and destroy a Server instance in the main method of the RestfulServer class.
>
> The last step to make the server up and running is to invoke that main method. In order to achieve that, the Exec Maven plugin is included and configured in the Maven POM file:

Section 4描述了怎样创建和销毁（没有说销毁）Server实例在RestfulServer的main方法中。

最后一步是调用main方法使服务运行起来。为了实现它，exec-maven-plugin被引入而且配置好了在pom文件中: 

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>1.5.0<version>
    <configuration>
        <mainClass>
          com.baeldung.cxf.jaxrs.implementation.RestfulServer
        </mainClass>
    </configuration>
</plugin>
```
> The latest version of this plugin can be found via this link.
> 
> In the process of compiling and packaging the artifact illustrated in this tutorial, the Maven Surefire plugin automatically executes all tests enclosed in classes having names starting or ending with Test. If this is the case, the plugin should be configured to exclude those tests:

在编译打包的过程中，the Maven Surefire plugin自动执行所有的类名以Test开头或结尾的类的测试。如果是这种情况，这个插件应该被配置为exclude这些测试: 

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <configuration>
    <excludes>
        <exclude>**/ServiceTest</exclude>
    </excludes>
    </configuration>
</plugin>
``` 

> With the above configuration, ServiceTest is excluded since it is the name of the test class. You may choose any name for that class, provided tests contained therein are not run by the Maven Surefire plugin before the server is ready for connections.
>
> For the latest version of Maven Surefire plugin, please check here. 下载最新插件
>
> Now you can execute the exec:java goal to start the RESTful web service server and then run the above tests using an IDE. Equivalently you may start the test by executing the command mvn -Dtest=ServiceTest test in a terminal.  

通过上述配置，ServiceTest被排除了，既然它是测试类的名字。你可以选择任何名字为那个类，类里边提供了测试不能在server准备好之前被Maven Surefire plugin运行。

现在你可以执行exec:java goal来启动RESTful web service服务器并且然后运行上述测试使用IDE. 一样地，你可以通过命令 mvn -Dtest=ServiceTest test 在命令行开启测试。 


## 7. Conclusion 总结 

> This tutorial illustrated the use of Apache CXF as a JAX-RS implementation. It demonstrated how the framework could be used to define resources for a RESTful web service and to create a server for publishing the service.
> 
> The implementation of all these examples and code snippets can be found in the [GitHub project](https://github.com/eugenp/tutorials/tree/master/apache-cxf/cxf-jaxrs-implementation). 

这篇文章没有太多的信息，对我来说有用的就是，我知道了webservice还可以支持Restful,跟Controller接口访问没有任何区别。






















<br/><br/><br/><br/>

