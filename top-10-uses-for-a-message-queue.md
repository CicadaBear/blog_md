# **Top 10 Uses For A Message Queue**  

Original [Top 10 Uses For A Message Queue](https://blog.iron.io/top-10-uses-for-message-queue/)  

![Geese love queues.](https://cicadabearblog.oss-cn-shenzhen.aliyuncs.com/10UsesMQ_Geese_Ducks.png)   
Geese love queues. (Image by [D.Hilgart](http://www.flickr.com/photos/david-hilgart/4142396713/))

We’ve been working with, building, and evangelising message queues for the last year, and it’s no secret that we think they’re awesome. We believe message queues are a vital component to any architecture or application, and here are ten reasons why:   

去年我们一直用消息队列，构建消息队列，还有传播消息队列，并且我们认为消息队列非常棒。我们相信消息队列是一个至关重要的组件对于任意的架构或者应用，并且这里有10个原因，为什么这样说：  

1. **Decoupling** It’s extremely difficult to predict, at the start of a project, what the future needs of the project will be. By introducing a layer in between processes, message queues create an implicit, data-based interface that both processes implement. This allows you to extend and modify these processes independently, by simply ensuring they adhere to the same interface requirements.   
   解耦，在项目开始的时候非常难预测，这个项目将来会需要什么。通过引入一层在处理层之间，消息队列创建了一个隐式的，基于数据的接口，两边的处理层都有实现。这样就可以允许我们独立的拓展修改这些处理层，通过简单的确保他们遵循相同的接口要求。

2. **Redundancy** Sometimes processes fail when processing data. Unless that data is persisted, it’s lost forever. Queues mitigate this by persisting data until it has been fully processed. The put-get-delete paradigm, which many message queues use, requires a process to explicitly indicate that it has finished processing a message before the message is removed from the queue, ensuring your data is kept safe until you’re done with it.  
   冗余，有些时候处理数据会出现失败的情况。除非数据是持久的，否则将永远丢失。队列通过保存数据直到数据被完全处理，使这个丢失数据的问题减轻了。这个put-get-delete范例，很多消息队列用的范例，需要一个程序来明确的表明它完成了消息的处理，在消息被移除之前，确保数据安全直到完成。  

3. **Scalability** Because message queues decouple your processes, it’s easy to scale up the rate with which messages are added to the queue or processed; simply add another process. No code needs to be changed, no configurations need to be tweaked. Scaling is as simple as adding more power.  
   可伸缩性，因为消息队列解耦了你的处理层，调高message加入到队列的速率和被处理的速率很简单；简单地再添加一个处理。没有代码需要被改，没有配置需要被调节。伸缩像添加更多power一样简单。 

4.  **Elasticity & Spikability** When your application hits the front page of Hacker News, you’re going to see unusual levels of traffic. Your application needs to be able to keep functioning with this increased load, but the traffic is anomaly, not the standard; it’s wasteful to have enough resources on standby to handle these spikes. Message queues will allow beleaguered components to struggle through the increased load, instead of getting overloaded with requests and failing completely. Check out our [Spikability blog post](http://blog.iron.io/2012/06/spikability-applications-ability-to.html) for more information about this.  
    弹性和高峰，当你的应用登上 Hacker News 头版的时候，你将会看到不同寻常级别的访问量。你的应用需要可以正常运行在增长的负载情况下，访问量不是标准的，是异常的。那会非常浪费，如果准备足够的资源来应对这些高峰期。消息队列将会允许被包围的组件来挣扎通过增长的负载，取代的是，高访问量过载，导致完全瘫痪。查阅我们的帖子来得到更多的相关信息。  

5. **Resiliency** When part of your architecture fails, it doesn’t need to take the entire system down with it. Message queues decouple processes, so if a process that is processing messages from the queue fails, messages can still be added to the queue to be processed when the system recovers. This ability to accept requests that will be retried or processed at a later date is often the difference between an inconvenienced customer and a frustrated customer.  
  弹性，当你的架构的一部分故障时，不需要跟着他瘫痪。消息队列解耦了程序，如果一个程序负责处理队列里的消息故障了，消息仍然可以被添加到队列来被处理，当系统恢复的时候。这个能接收请求之后重试处理的能力的能力，一般是一个不方便的用户和一个挫败的用户的不同。  

6. **Delivery Guarantees** The redundancy provided by message queues guarantees that a message will be processed eventually, so long as a process is reading the queue. On top of that, IronMQ provides an only-delivered-once guarantee. No matter how many processes are pulling data from the queue, each message will only be processed a single time. This is made possible because retrieving a message “reserves” that message, temporarily removing it from the queue. Unless the client specifically states that it’s finished with that message, the message will be placed back on the queue to be processed after a configurable amount of time.  
  交付保证，这个消息队列提供的冗余保证了一个消息最终会被处理，只要有一个程序从队列里读出。除此之外，IronMQ提供了只交付一次的保证。不管多少个程序在从队列里读数据，每一个消息被处理一次。这能变的可行是因为获取了一个消息预定，预定了那条消息，临时的从队列里移除了。除非，客户端特别说明这条消息是完成的，否则这条消息会被重新放回队列等待处理，在一段可配置的时间后。  

7. **Ordering Guarantees** In a lot of situations, the order with which data is processed is important. Message queues are inherently ordered, and capable of providing guarantees that data will be processed in a specific order. IronMQ guarantees that messages will be processed using FIFO (first in, first out), so the order in which messages are placed on a queue is the order in which they’ll be retrieved from it.  
  顺序的保证，在很多情况下，处理数据的顺序非常重要。消息队列天生的有序的，并且有提供数据会被以特定的顺序处理的保证的能力。IronMQ保证消息会被处理使用先进先出的原则，所以消息被放进队列里的顺序就是他们被从队列里获取的顺序。  

8. **Buffering** In any non-trivial system, there are going to be components that require different processing times. For example, it takes less time to upload an image than it does to apply a filter to it. Message queues help these tasks operate at peak efficiency by offering a buffer layer–the process writing to the queue can write as fast as it’s able to, instead of being constrained by the readiness of the process reading from the queue. This buffer helps control and optimize the speed with which data flows through your system.  
  缓冲，在任何有用的系统，这里会有组件需要不同的处理时间。比如，它上传一张图片比对一张图片用过滤器用更少的时间。消息队列帮助这些任务操作达到峰值效率通过提供缓冲层 - 程序写入队列，可以尽可能快的写入，而不是被限制在准备就绪为从队列读出数据。这个缓冲帮助了控制和优化数据流通过你的系统的速度。  
  
9. **Understanding Data Flow** In a distributed system, getting an overall sense of how long user actions take to complete and why is a huge problem. Message queues, through the rate with which they are processed, help to easily identify under-performing processes or areas where the data flow is not optimal.  
  理解数据流，在分布式系统中，获取一个用户动作需要多长时间完成和为什么是一个巨大的问题的整体观。消息队列，通过数据的处理速率，容易定位出表现不佳的程序或者地方，那里数据流没有优化。  

10. **Asynchronous Communication** A lot of times, you don’t want to or need to process a message immediately. Message queues enable asynchronous processing, which allows you to put a message on the queue without processing it immediately.   For long-running [API](http://blog.dreamfactory.com/soap-vs-rest-apis-understand-the-key-differences/) calls, [SQL](https://www.sqlbot.co/) reporting queries, or any other operation that takes more than a second, consider using a queue.   Queue up as many messages as you like, then process them at your leisure.  
  异步交流，很多时候，你不想或者不需要立即处理消息。消息队列使能够异步处理，这允许你把消息放在队列，而不立即处理。 对应需要长时间运行的API调用，SQL报表查询，或者任何其他操作需要超过一秒的时间考虑使用队列。 队列化尽可能多的消息，然后放在空闲时间处理。  

We believe these ten reasons make queues the best form of communication between processes or applications. We’ve spent a year building and learning from IronMQ, and our customers are doing amazing things with message queues. Queues are the key to the powerful, distributed applications that can leverage all the power that the cloud has to offer.  

我们相信这10个理由使队列成为了程序间应用间最好的交流形式。我们用了一年的时间来构建和学习IronMQ,并且我们的客户在做惊人的事情通过消息队列。队列是强大的分布式应用的关键，它可以利用云提供的所有能力。 

If you’d like to get started with an efficient, reliable, and hosted message queue today, [check out IronMQ](http://iron.io/products/mq?rc=blog_mq_t10).  Read about the difference between IronMQ and Amazon SQS if you’re considering both.  If you’d like to connect with our engineers about how queues could fit into your application, they’re always available at get.iron.io/chat.  

***
UPDATE: We posted a related article recently on the uses of a worker system titled Top 10 Uses of IronWorker (although the uses can apply to any worker system/task queue). They often work in concert with message queues and are used for background processing, schedule jobs, event processing, as a mobile compute cloud, and for many other uses.






  


   
   


<br><br><br><br><br><br>