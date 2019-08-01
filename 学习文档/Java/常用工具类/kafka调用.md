## kafka调用新手篇

### 写在前面

​	结合springBoot项目中使用kafka，使用的是net.sf.json的byte[]数据传输，如果使用Gson的byte[]数据传输，json格式中会带有转义字符

​	**<个人习惯用net.sf.json解析数据，net.sf.json解析不了Gson的带转义字符的json数据>。**

---

#### kafka基本术语

​	**<作为阅读即可，没实际卵用，面试还老爱被问的东西>**

​	Producer
​		生产者，是发送消息的对象

​	Consumer
​		消费者，是订阅消息和处理消息的对象

​	Topic
​		主题，用于消息的分类，也就是一个标签，可以看作是一个频道，可以被多个消费者订阅

​	Broker

​		代理，kafka集群中的每一个服务器就是一个代理（Broker），消费者可以订阅一个或者多个主题（Topic），消费已经发布的主题

​	**一句话原理：生产者产生数据，把数据绑定一个topic发送到kafka集群中，消费者看到订阅的topic有数据，就去获取对应topic的数据。**

---

### 一、pom配置

~~~xml
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>

        <!-- kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <!--kafka end-->

        <!-- json依赖 -->
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <classifier>jdk15</classifier>
            <version>2.4</version>
        </dependency>
        <dependency>
            <groupId>net.sf.ezmorph</groupId>
            <artifactId>ezmorph</artifactId>
            <version>1.0.6</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
        </dependency>
        <!-- json依赖 end -->
~~~

---

### 二、yml配置篇

​	此处需要注意send（）方法发送的数据格式，本片以byte[]格式为讲解，String格式等只需要修改 ByteArrayDeserializer 为 String的 StringSerializer 数据序列化即可

~~~yaml
spring:
  kafka: #kafka消息配置
     bootstrap-servers: 10.7.130.30:9092,10.7.130.31:9092,10.7.130.36:9092
     consumer:
       group-id: qnb100
       key-deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
       value-deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
     producer:
       key-serializer: org.apache.kafka.common.serialization.ByteArraySerializer
       value-serializer: org.apache.kafka.common.serialization.ByteArraySerializer
~~~

---

### 三、代码篇

#### 消息生产者

1. Controller层

~~~java
package com.example.demo.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * @program: demo
 * @Date: 2019/6/25 11:18
 * @Author: Mr.Yang
 * @Description:
 */
@RestController
public class DemoController {

    @Autowired
    KafkaDemo kafkaDemo;

    @RequestMapping(value = "/demo" ,method = RequestMethod.GET)
    private void demo(){
        MessageModel messageModel = new MessageModel();
        List<CustomerGroupBo> customerGroupBoList = new ArrayList<>();
        CustomerGroupBo customerGroupBo = new CustomerGroupBo();
        customerGroupBo.setCustomerno("12312312");
        customerGroupBoList.add(customerGroupBo);
        kafkaDemo.sendMessage(customerGroupBoList);
    }
}

~~~

2. Service层

   slup_customer_groupInfo 这个是消息主题(topic)

~~~java
package com.example.demo.demo;

import net.sf.json.JSONArray;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;

import java.util.List;

/**
 * @program: demo
 * @Date: 2019/6/25 10:59
 * @Author: Mr.Yang
 * @Description:
 */
@Component
public class KafkaDemo {

    @Autowired
    private KafkaTemplate kafkaTemplate;

    public void sendMessage(List<CustomerGroupBo> customerGroupBoList){

        JSONArray jsonArray = JSONArray.fromObject(customerGroupBoList);
        System.out.println("推送数据为-----------------" + jsonArray);

        try {
            ListenableFuture<?> future = kafkaTemplate.send("slup_customer_groupInfo", jsonArray.toString().getBytes());
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

#### 消息消费者

~~~java
package com.example.demo.demo;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import net.sf.json.JSONArray;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.UUID;

/**
 * @program: demo
 * @Date: 2019/6/25 15:39
 * @Author: Mr.Yang
 * @Description:
 */
@Component
public class KafkaConsumer {
    private Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd HH:mm:ss").create();

    /**
     * 监听test主题,有消息就读取
     */
    @KafkaListener(topics = {"slup_customer_groupInfo"})
    public void consumer(byte[] bytes){
        System.out.println("---------------------------------来了老弟");
        String uuid = UUID.randomUUID().toString().replace("-", "");
        System.out.println("流水号为" + uuid + "，kafka消费团险个人资料数据开始"+bytes);
        String jsonContent = new String(bytes);
        System.out.println("消费数据："+jsonContent);
        JSONArray jsonArray = JSONArray.fromObject(jsonContent);
        System.out.println(jsonArray);
    }
}

~~~

---

## kafka进阶篇

### 一、kafka介绍

​		Kafka是最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编写，Linkedin于2010年贡献给了Apache基金会并成为顶级开源 项目。

### 二、kafka特性

\- 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group[^consumer]对partition进行consume操作。

\- 可扩展性：kafka集群支持热扩展

\- 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失

\- 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）

\- 高并发：支持数千个客户端同时读写

### 三、应用场景

\- 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。

\- 消息系统：解耦和生产者和消费者、缓存消息等。

\- 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。

\- 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。

\- 流式处理：比如spark streaming和storm

\- 事件源



https://blog.csdn.net/sun123_123/article/details/86137141