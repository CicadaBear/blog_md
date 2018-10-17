# **Message Queues & You – 12 Reasons to Use Message Queuing**

Original [Message Queues & You – 12 Reasons to Use Message Queuing](https://stackify.com/message-queues-12-reasons/)  

At Stackify, we receive a lot of data from your apps to our Retrace APIs. We queue all of that data as soon as it gets to us, and then we use separate background services to process the data. Message queues help a lot with ensuring data is never lost, traffic spikes, etc. We decided to put together a list of why you should be using message queues!  

在Stackify，我们收到了大量的数据，从你们的App到我们的追溯接口。我们排列了数据，一旦数据到了我们这，并且然后，我们使用不同的后台服务来处理这些数据。消息队列帮助我们确保数据从没有丢失，数据高峰期，等待，起了很大帮助。我们决定把为什么我们使用消息队列做成一个列表。  

## **1. Redundancy via Persistence 通过持久化冗余**  

Redundancy is one of the most obvious advantages to message queues. Application crashes, timeouts, errors in your code, and other problems are just part of the norm. This is especially true in applications that process millions or billions of transactions per month.  

冗余是消息队列最明显的几大优势之一。应用崩溃，超时，错误在你的代码中，还有其他问题，都很常见。这些问题尤其常见在处理百万或者十亿级别的事务时，每个月。

Queues help with redundancy by making the process that reads the message confirm that it completed the transaction and it is safe to remove it. If anything fails, worst case scenario, the message is persisted to storage somewhere and won’t be lost. It can be reprocessed later.  

队列通过使程序读消息来确认完成了事务，并且移除这个事务是安全的，来实现的冗余。如果有任何失败，最坏的情况，这条消息被保存在了存储中，并且不会丢失。这条消息可以在之后重新处理。  

## **2. Traffic Spikes “交通”高峰**  

You don’t always know exactly how much traffic your application is going to have. For example, at Stackify we receive billions of messages a month. We have no way to know what our clients are going to send us. By queuing the data we can be assured the data will be persisted and then be processed eventually, even if that means it takes a little longer than usual due to a high traffic spike.   

你不能确切的知道你的应用将会有多少请求量。比如，在 Stackify 我们收到了有一个月十亿级别的消息。我们无从得知我们的用户要发给我们什么。通过队列化数据，我们可以确保数据将会被保存，并且之后会最终处理, 即使这意味着处理时间比往常长一些，由于交通高峰的缘故。  

## **3. Improve Web Application Page Load Times 提高Web应用页面加载时间**  

Queues can be useful in web applications to do complex logic in a background thread so the request can finish for the user quickly. If someone places an order on your website, that could involve a lot of different things that have to happen. You can do the minimum and return success to your user and kick off the rest of them in a background thread to finish up, without using a full message queuing system and background apps. Most programming languages have ways to do this now. Examples: Resque, Hangfire, etc.  

队列会很有用当web应用在后台线程做复杂的逻辑处理时，会使用户的请求很快结束。如果有人在网站下一个订单，这可能会涉及很多东西要处理。你可以做最少，并且返回成功给你的用户并且开始在后台线程做剩余的事来完成订单，不用使用a full message queuing system and background apps 。 现在大多数的编程语言可以做到这些。例如，Resque, Hangfire, etc.  

## **4. Batching for Efficiency 为高效率分批处理**  

Batching is a great reason to use message queues. It is much more efficient to insert 100 records into a database at a time instead of 1 at a time, 100 times. We insert a lot of data into elasticsearch and SQL Server. Batching helps us optimize their performance by tuning the size of the transactions.   

分批是一个使用消息队列的很大原因。在数据库一次插入100条，比一次插入1条，插100次，更高效。我们向elasticsearch和SQL Server，插入多非常多的数据。分批处理优化了性能，通过调节事务的大小。  

## **5. Asynchronous Messaging 异步消息**

Queues can be great in scenarios where your application needs something done but doesn’t need it done now, or doesn’t even care about the result. Instead of calling a web service and waiting for it to complete, you can write the message to a queue and let the same business logic happen later. Queues are an excellent way to implement an asynchronous programming pattern.  

队列可以做到非常好在一下的场景中，你的应用需要完成一些事，但是不需要现在完成，或者不关心处理结果。取而，调用web service并且等待它完成，代之，你可以把消息写入队列，让相同大的业务逻辑处理稍后执行。队列是一种非常棒的方式来实现异步处理模式。  

## **6. Decouple by Using Data Contracts 通过使用数据协约解耦**  

By using a queue between different parts of your software you can decouple the hard dependencies. The format of the message in the queue becomes your data contract and anything that knows how to read that message format can be used to process the transaction. This could be useful for parts of your code that are even written in different programming languages.  

通过在软件的不同部分之间使用队列，你可以解耦强依赖关系。 队列里的消息格式，变成了你的数据协约，并且还有跟怎样读取消息使消息可以被用于处理事务的所有东西都是协约。 如果你的代码使用的是不用的编程语言，这个方法很有用。  

## **7. Transaction Ordering and Concurrency Challenges 交易订单和并发挑战**  

If 1000 people are placing an order on your website at one time, that could create some problems with concurrency and ensuring that the first order in finishes first. By queuing them up, you could then guarantee their order and control how many are even processed concurrently.  

如果有1000个人在你的网站同时下单，这会造成一些并发问题，和确保第一条订单第一个完成的问题。通过把订单队列化，你可以保证他们的订单并且甚至控制多少个订单同时处理。  

## **8. Improve Scalability 提高可伸缩性**  

Message queues enable you to decouple different parts of your application and then scale them independently. Using Azure, AWS, or other hosting solutions you could even dynamically scale that background service based on CPU usage or other metrics. Message queues can help a lot with scalability and elasticity.  

消息队里使你可以解耦你应用的不同部分，并且之后可以分别单独“扩展”他们。使用Azure, AWS或者其他的主机解决方案，你甚至可以动态的拓展后台服务基于CPU使用量和其他一些指标。 消息队列对可伸缩性和弹性有非常大的帮助。  

## **9. Create Resiliency 创建弹性**  

By breaking your app up and separating different components by queues, you inherently create more resiliency. Your website can still be operational even if part of the back end processing of the order is slightly delayed. At Stackify, we design our incoming APIs to do as little as possible beyond queuing the data to ensure that nothing else can bring down the ability to receive data. Even if SQL Server is down, we want to be able accept data!  

通过拆分你的应用并且通过队列分为不同的组件，你自然而然的创建了更多的弹性。你的网站可以仍然是运行的，即使部分后台订单处理稍微延时了。在Stackify，我们设计了我们的入口API，本着除了把数据放入队列，之外尽可能少的做其他处理，以此来确保没有其他，会降低接收数据的能力。即使 SQL Server 挂了，我们仍然想要接收数据。 

## **10. Guarantee Transaction Occurs Once 确保事务只发生一次**  

Message queues are designed to be transactional. When you read a message off of a queue, you have to tell it where you completed processing the message before it is completely removed from the queue. This helps to ensure that a transaction only occurs once.  

消息队列被设计为事务型的。当你从队列中读出一条消息时，在这条消息被移除之前，你必须告诉它，在哪你完成了处理。这样会确保一个事务（交易）只会出现一次。    


## **11. Break Larger Tasks into Many Smaller Ones 分解大任务为小任务** 

Another good use for queues is breaking up a larger task into lots of little pieces and then queuing them all up. We have a good example of this at Stackify. If you edit a monitoring template for a group of servers, we then need to update the monitors on every server that uses that template. We can queue up a message for every server and potentially do them concurrently as smaller operations.  

另外一个队列的好的用法，是分解大任务为多个小块，然后把这些小块队列化。我们有一个很好的例子在 Stackify。如果你编辑一个监控模板对于一组服务器，然后我们需要更新监视器在每一个用到这个模板的服务器上。我们可以队列化一条消息为每一个服务器，并且潜在的同时做他们作为小任务。 

## **12. Monitoring 监控**

Message queuing systems enable you to monitor how many items are in a queue, the rate of processing messages, and other stats. This can be very helpful from an application monitoring standpoint to keep an eye on how data is flowing through your system and if it is getting backed up.  

消息队列系统使你能够监控多项在一个队列中，处理消息的速率，还有其他的状态。这些可能非常有用，从应用监控的视角，来保持监视数据是怎样流经你的系统的，并且如果数据被备份起来。  

## **Does every app need message queues? 每个应用都需要消息队列吗？**

No, of course not. Like most things in life, there is a place for everything.  

不，当然不是。就想生活中的大多数事情一样, 凡事都有一个地方。

However, I would suggest at least thinking about improving the performance of your web applications by using background tasks to offload some long running transactions.  

然而，我会至少建议考虑通过使用后台任务来卸下长时间运行的事务来提高web应用的性能。 

## **Alternative “In app” Background Processing 可选的应用内部后台处理**

You can consider Hangfire, Resque and other similar projects for this. Those projects act as sort of mini queuing systems and can persist the jobs for resiliency. They run entirely in your web application so it doesn’t require standing up additional servers or anything.  

你可以考虑Hangfire, Resque还有其他的类似项目用来后台处理的。这些项目扮演着有点像mini 队列系统，并且可以保存这些工作为了弹性。他们完全运行在web应用内部，所有不需要建立一个外部服务器等。  

Another option without the resiliency is using fire and forget tasks in your code. But if your app crashes before it finishes or some sort of error occurs, that operation would be lost.  

另外一个可选项不会提高弹性的，是使用fire或者forget 任务在代码中。但是，如果你的引用崩溃了或者出现了错误在它完成之前，这个操作将会丢失。 

ASP.NET has a handy way to do this. If you do it this way, the framework will even delay stopping your app until your code finishes. But if your app crashes, it would be lost. Using Hangfire would provide more functionality and the resiliency.  

ASP.NET有一个便利的方法来做这些。如果你使用这个方法，这个框架将甚至会延迟停止你的应用直到你的代码运行完成。但是，如果你的应用崩溃了，数据将会丢失。使用 Hangfire 将会提供更多的功能和弹性。  

```
HostingEnvironment.QueueBackgroundWorkItem(x =>
	{
		//do some stuff here, like maybe send an email?
		SmtpClient smtp = new SmtpClient();
		smtp.Send();
	});
```  

You can read more about HostingEnvironment.QueueBackgroundWorkItem at [MSDN](https://blogs.msdn.microsoft.com/webdev/2014/06/04/queuebackgroundworkitem-to-reliably-schedule-and-run-background-processes-in-asp-net/).  

## **Message Queues Conclusion 消息队列总结**  

Message queues are a very critical component to our platform at Stackify. We queue and process billions of messages a month. For us, it would be hard to imagine not having message queues!  

消息队列是一个非常重要的组件对于我们的平台来说在Stackify。我们排列处理十亿级别的消息一个月。对我们来说，很难想象没有消息队列。  

It can be a lot of work to stand up RabbitMQ, MSMQ or some other message queuing service. Although, Azure and AWS make it easier with their hosted offerings. We use Azure Service Bus ourselves and it has worked well for us.  

还有一些工作，才能让RabbitMQ，MSMQ或者其他的消息队列服务，跑起来。但是， Azure and AWS 使它更简单了通过使用它们提供的服务。我们自己就使用Azure Service Bus，它工作的非常好。  

How do you use message queues? Have any other great uses or tips? Comment below!













<br/><br/><br/><br/><br/><br/>