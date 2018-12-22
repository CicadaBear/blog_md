# Spring Kafka – Multi-threaded Message Consumption

[original](https://howtoprogram.xyz/2016/09/25/spring-kafka-multi-threaded-message-consumption/)

> As mentioned in [previous article](https://howtoprogram.xyz/2016/09/23/spring-kafka-tutorial/), Spring Kafka provides two MessageListenerContainer implementations: KafkaMessageListenerContainer and ConcurrentMessageListenerContainer. The KafkaMessageListenerContainer provides ability to consume messages from Kafka topics in a single thread while the ConcurrentMessageListenerContainer allows us to consume messages in multi-threaded style. In this article, we will get to know more detail about multi-threaded message consumption by using the second implementation, the ConcurrentMessageListenerContainer.

正如在上篇文章中提到的，Spring Kafka提供了2个MessageListenerContainer的实现: KafkaMessageListenerContainer 和 ConcurrentMessageListenerContainer. KafkaMessageListenerContainer提供了从Kafka topic中单线程消费消息的能力，然而ConcurrentMessageListenerContainer 使我们能够以多线程消费消息。在这篇文章中，我们将会了解更多关于多线程消息消费的细节，通过使用第二个实现，ConcurrentMessageListenerContainer。 

## 1. ConcurrentMessageListenerContainer 

> Here is the general class diagram of the MessageListenerContainer and its children.  

这个是MessageListenerContainer和它的子类的简单类图。

![MessageListener Class Diagram](https://raw.githubusercontent.com/cicadabear/blog_md/master/images/Message-Listener-Class-Diagram.png)

### 1.1. KafkaMessageListenerContainer 

> Firstly, let’s take a quick look at the KafkaMessageListenerContainer class. Here is one of its constructor: 

首先，我们快速看下KafkaMessageListenerContainer类，这里是它的一个构造方法: 

```
public KafkaMessageListenerContainer(ConsumerFactory<K, V> consumerFactory,
                    ContainerProperties containerProperties)
```

> The constructor require a **ConsumerFactory** object which includes information to create Apache Kafka consumer and a **ContainerProperties** object which includes information about the topics consumed by the consumer. 

构造方法需要一个**ConsumerFactory**对象，这个对象包含了创建Apache Kafka consumer的信息，还需要一个**ContainerProperties**，这个对象包含了consumer消费topic的信息。 

> The second constructor is: 

第二个构造函数: 

```
public KafkaMessageListenerContainer(ConsumerFactory<K, V> consumerFactory,
                    ContainerProperties containerProperties,
                    TopicPartitionInitialOffset... topicPartitions)
``` 

> This constructor is used by the ConcurrentMessageListenerContainer and requires information about partition and offset of the topic. 

这个构造函数被ConcurrentMessageListenerContainer使用，并且需要topic的partition和offset的信息。 

### 1.2. ConcurrentMessageListenerContainer 

> From the class diagram, we can see that the ConcurrentMessageListenerContainer has a property concurrency that represents the number of concurrent message containers (Apache Kafka consumers) to consume the messages from Apache Kafka topic. We also see the relationship between it and the KafkaMessageListenerContainer. So, if the number concurrency is 3, then it will create 3 KafkaMessageListenerContainer objects.  

从类图中我们可以看到，ConcurrentMessageListenerContainer有一个属性concurrency，这个属性代表了concurrent message containers（其实是 Apache Kafka consumers）的数量。我们也可以看到这个属性和KafkaMessageListenerContainer的关系。So, 如果concurrency是3，将会创建3个KafkaMessageListenerContainer对象。 

> One important thing we should remember when we work with Apache Kafka is the number of consumers in the same consumer group should be less than or equal the number of partitions in the consumed topic. Otherwise, the exceedable consumers will not be received any messages from the topic.  

一个重要的事需要我们记住，当使用Apache Kafka的时候，一个group的消费者的数量应当小于或者等于topic分区的数量。不然，超过的消费者不会从topic中接受任何消息。 

> With Spring Kafka, if the concurrency is greater than number of the topic partitions, then the concurrency will be  will be adjusted down such that each container will get one partition.  

使用Spring Kafka, 如果concurrency大于topic partitions的数量，然后concurrency将会被调低，直到，每个container得到一个分区。 

> If we have 3 partitions and the concurrency is 3, then each container will get 1 partition. If we have 4 partitions, then 1 container will get 2, and the 2 remain ones will get 1 partition for each. 

如果我们有3个分区，并且concurrency是3，就是每个container将会得到一个分区。如果我们有4个分区，就是1个container得到2个分区，剩下的两个container各得到一个分区。 

## 2. Multi-threaded Message Consumption Example 例子

Here is the detail of the example. 

* We have a topic: SpringKafkaTopic with 3 partitions.
* A ConcurrentMessageListenerContainer object with the concurrency = 3
* A KafkaTemplate, provided by Spring Kafka, to send the messages to the topic.

### 2.1. Preparations 准备

* Apache Kafka 0.9/0.10 broker installed on local or remote. You can refer to this link for setting up.
* JDK 8 installed on your development PC.
* IDE (Eclipse or IntelliJ)
* Maven 

### 2.2. Source Code Structure 代码结构

The source code was added to the [Github](https://github.com/howtoprogram/apache-kafka-examples/tree/master/spring-kafka-multi-threaded-consumption). You can clone from it or download the zip file. The zip file contains multiple examples of Spring Kafka. For this example, check the **spring-kafka-multi-threaded-consumption** sub project. 

To import in to Eclipse. 

* Menu File –> Import –> Maven –> Existing Maven Projects
* Browse to your source code location 

Click Finish button to finish the importing 

### 2.3. Spring Kafka dependencies 

The only dependency we will need is spring-kafka 

```
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>1.1.0.RELEASE</version>
</dependency>
```

However, this example is built on Spring Boot, and we also need to run JUnit tests, the dependencies are more as following: 

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.howtoprogram</groupId>
    <artifactId>spring-kafka-example1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-kafka-example1</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.0.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>1.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <version>1.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Brixton.SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### 2.4. Classes Descriptions 类描述

> We will separate the configuration of the consumer and producer into different classes for easy to get. 

我们将会分离consumer和producer的配置到不同类中。

#### 2.4.1. KafkaConsumerConfig 

```
package com.howtoprogram.kafka;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

@Configuration
@EnableKafka
public class KafkaConsumerConfig {
	@Bean
	KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
		ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
		factory.setConsumerFactory(consumerFactory());
		factory.setConcurrency(3);
		factory.getContainerProperties().setPollTimeout(3000);
		return factory;
	}

	@Bean
	public ConsumerFactory<String, String> consumerFactory() {
		return new DefaultKafkaConsumerFactory<>(consumerConfigs());
	}

	@Bean
	public Map<String, Object> consumerConfigs() {
		Map<String, Object> propsMap = new HashMap<>();
		propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
		propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
		propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "100");
		propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");
		propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
		propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, "group1");
		propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		return propsMap;
	}

	@Bean
	public Listener listener() {
		return new Listener();
	}
}
```

> We have specified the configuration for the consumer in the **consumerConfigs()** method. The **concurrency=3** property is set in the **kafkaListenerContainerFactory()** method.  
Note that we have configured a listener bean. Let’s take a look at it in the next section.

我们为consumer指定了配置在consumerConfigs()中。设置了concurrency=3在kafkaListenerContainerFactory()方法中。  
注意我们配置了一个listener bean，我们下一章看一下它。 

#### 2.4.2. Listener.java 

```
package com.howtoprogram.kafka;

import java.util.concurrent.CountDownLatch;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.annotation.TopicPartition;

public class Listener {
    public CountDownLatch countDownLatch0 = new CountDownLatch(3);
    public CountDownLatch countDownLatch1 = new CountDownLatch(3);
    public CountDownLatch countDownLatch2 = new CountDownLatch(3);

    @KafkaListener(id = "id0", topicPartitions = { @TopicPartition(topic = "SpringKafkaTopic", partitions = { "0" }) })
    public void listenPartition0(ConsumerRecord<?, ?> record) {
        System.out.println("Listener Id0, Thread ID: " + Thread.currentThread().getId());
        System.out.println("Received: " + record);
        countDownLatch0.countDown();
    }

    @KafkaListener(id = "id1", topicPartitions = { @TopicPartition(topic = "SpringKafkaTopic", partitions = { "1" }) })
    public void listenPartition1(ConsumerRecord<?, ?> record) {
        System.out.println("Listener Id1, Thread ID: " + Thread.currentThread().getId());
        System.out.println("Received: " + record);
        countDownLatch1.countDown();
    }

    @KafkaListener(id = "id2", topicPartitions = { @TopicPartition(topic = "SpringKafkaTopic", partitions = { "2" }) })
    public void listenPartition2(ConsumerRecord<?, ?> record) {
        System.out.println("Listener Id2, Thread ID: " + Thread.currentThread().getId());
        System.out.println("Received: " + record);
        countDownLatch2.countDown();
    }
}
```

CountDownLatch是我第一次见到值得说一下，指的是CountDownLatch.coutDown()3次，CountDownLatch.await(long timeout,
            TimeUnit unit)才会释放,并返回true,在超时时间内。下文测试用例章节会看到await用法。
            

> We have used **@KafkaListener** to define the listeners for different partitions of the topic SpringKafkaTopic. The countDownLatch properties are used mainly for testing purpose. 

#### 2.4.3. KafkaProducerConfig.java 

> This class define producer configuration which use KafkaTemplate to produce the messages to the topic. 

```
package com.howtoprogram.kafka;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

@Configuration
@EnableKafka
public class KafkaProducerConfig {
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<String, String>(producerFactory());
    }
}
```

#### 2.4.4. kafka-topics.sh 

> The script to create the topic SpringKafkaTopic with partitions. Note that if we don’t create the topic, it will be created automatically but with only 1 partition, and we can not test correctly this example. 

这个脚本是用来创建SpringKafkaTopic和指定数量分区的。注意，如果我们不创建这个topic, 它将会自动创建，但是只会创建一个分区，而且我们不能正确测试这个例子。 

```
#cd $APACHE_KAFKA_HOME
 ./bin/kafka-topics.sh --create --zookeeper localhost:2181  --replication-factor 1 --partitions 3 --topic SpringKafkaTopic
```

> The kafka-topics.bat is used to create the topic on Windows environment.

windows环境下用kafka-topics.bat

#### 2.4.5. SpringKafkaMultipleConsumptionTests 

> The unit test class of our example. 

我们例子的单元测试类。

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaMultipleConsumptionTests {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    @Autowired
    private Listener listener;

    @Test
    public void contextLoads() throws InterruptedException {

        for (int i = 0; i < 9; i++) {
            ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send("SpringKafkaTopic",
                    "Messsage:" + i);
            future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
                @Override
                public void onSuccess(SendResult<String, String> result) {
                    System.out.println("Sent message: " + result);
                }

                @Override
                public void onFailure(Throwable ex) {
                    System.out.println("Failed to send message");
                }
            });
        }

        assertThat(this.listener.countDownLatch0.await(60, TimeUnit.SECONDS)).isTrue();
        assertThat(this.listener.countDownLatch1.await(60, TimeUnit.SECONDS)).isTrue();
        assertThat(this.listener.countDownLatch2.await(60, TimeUnit.SECONDS)).isTrue();
    }

}
```

> We have injected the kafkaTemplate to send 9 messages to the topic SpringKafkaTopic, and we expect that each of 3 container will receive 3 messages. 

我们注入kafkaTemplate，发了9条消息到topic SpringKafkaTopic, 并且我们期望3个container中每一个接收3条消息。 

#### 2.4.6 Run the example 

**Step 1.** Make sure the Apache Kafka is running on localhost:9092  

**Step 2.** Run the  **kafka-topics.sh** or  **kafka-topics.bat** to create the topic with 3 partitions.  

**Step 3.** Open the **SpringKafkaMultipleConsumptionTests.java**, **Right click** –> **Run As** –> **JUnit Test** or use the shortcut: Alt+Shift+x, t to start the test.  

> The output on the console should be similar to: 

你的控制台输出应该跟下边的相似: 

```
Listener Id2, Thread ID: 17
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 2, offset = 6, CreateTime = 1474782637845, checksum = 386545754, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:1)
Listener Id1, Thread ID: 18
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 1, offset = 6, CreateTime = 1474782637845, checksum = 2382588384, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:2)
Listener Id2, Thread ID: 17
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 2, offset = 7, CreateTime = 1474782637845, checksum = 1734397141, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:4)
Listener Id0, Thread ID: 15
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 0, offset = 6, CreateTime = 1474782637839, checksum = 240789682, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:0)
Listener Id2, Thread ID: 17
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 2, offset = 8, CreateTime = 1474782637846, checksum = 1564416966, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:7)
Listener Id1, Thread ID: 18
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 1, offset = 7, CreateTime = 1474782637845, checksum = 275250243, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:5)
Listener Id0, Thread ID: 15
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 0, offset = 7, CreateTime = 1474782637845, checksum = 4177811830, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:3)
Listener Id0, Thread ID: 15
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 0, offset = 8, CreateTime = 1474782637846, checksum = 708324176, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:6)
Listener Id1, Thread ID: 18
Received: ConsumerRecord(topic = SpringKafkaTopic, partition = 1, offset = 8, CreateTime = 1474782637846, checksum = 3447719511, serialized key size = -1, serialized value size = 10, key = null, value = Messsage:8)
```

## 3. Summary 总结  

> We have learned about multi-threaded message consumption by using ConcurrentMessageListenerContainer in the Spring for Apache Kafka. This wrapper of Spring Kafka facilitates the using of multi-threaded consumer model in Apache Kafka which improve the performance in message consumer. If you’re looking for the native approaches, you can refer to my previous post: Create Multi-threaded Apache Kafka Consumer 

我们已经学习过多线程消息消费通过ConcurrentMessageListenerContainer in the Spring for Apache Kafka。这个Spring Kafka的包装使多线程消费者模型的使用变得容易，多线程消费者模型提升了消息消费的性能。如果你在寻找原生的实现方法，你可以参考我之前的帖子，[Create Multi-threaded Apache Kafka Consumer](https://howtoprogram.xyz/2016/05/29/create-multi-threaded-apache-kafka-consumer/)。 

> Below are other articles related to the Apache Kafka for your reference: 

以下使其他跟Apache Kafka相关的文章供你参考: 

[Spring Kafka Tutorial – Getting Started with Spring for Apache Kafka](https://howtoprogram.xyz/2016/09/23/spring-kafka-tutorial/)

[Apache Kafka Tutorial](https://howtoprogram.xyz/big-data-technologies/apache-kafka-tutorial/)

[Getting started with Apache Kafka 0.9](https://howtoprogram.xyz/2016/04/30/getting-started-apache-kafka-0-9/)

[Apache Kafka 0.9 Java Client API Example](https://howtoprogram.xyz/2016/05/02/apache-kafka-0-9-java-client-api-example/)

[Using Apache Kafka Docker](https://howtoprogram.xyz/2016/07/21/using-apache-kafka-docker/)

[Create Multi-threaded Apache Kafka Consumer](https://howtoprogram.xyz/2016/05/29/create-multi-threaded-apache-kafka-consumer/)

[Apache Kafka Command Line Interface](https://howtoprogram.xyz/2016/07/08/apache-kafka-command-line-interface/)

[Write An Apache Kafka Custom Partitioner](https://howtoprogram.xyz/2016/06/04/write-apache-kafka-custom-partitioner/)

[How To Write A Custom Serializer in Apache Kafka](https://howtoprogram.xyz/2016/06/06/write-custom-serializer-apache-kafka/)

[Apache Kafka Connect Example](https://howtoprogram.xyz/2016/07/10/apache-kafka-connect-example/)

[Apache Kafka Connect MQTT Source Tutorial](https://howtoprogram.xyz/2016/07/30/apache-kafka-connect-mqtt-source-tutorial/)

[Apache Flume Kafka Source And HDFS Sink Tutorial](https://howtoprogram.xyz/2016/08/06/apache-flume-kafka-source-and-hdfs-sink/)





<br/><br/><br/><br/>
