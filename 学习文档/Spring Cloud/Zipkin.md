## Zipkin

### 一、没用的背景

​	微服务架构是一个分布式架构，微服务系统按业务划分服务单元，一个微服务系统往往有很多个服务单元。由于服务单元数量众多，业务的复杂性较高，如果出现了错误和异常，很难去定位。主要体现在一个请求可能需要调用很多个服务，而内部服务的调用复杂性决定了问题难以定位。所以在微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题能够快速定位的目的。

​	启动

~~~
docker run -ditp 0.0.0.0:9411:9411 -e zipkin.torage.type=elasticsearch -e zipkin.torage.elasticsearch.hosts=http://localhost:9200 -e zipkin.torage.elasticsearch.index=zipkin -e ES_JAVA_OPTS="-Xms256m -Xmx256m" openzipkin/zipkin
~~~

---

### 二、数据存储方式

​	Zipkin 提供了可插拔数据存储方式：In-Memory、MySql、Cassandra 以及 Elasticsearch。接下来的测试为方便直接采用 In-Memory 方式进行存储，生产推荐 Elasticsearch。

---

### 三、需要的依赖及操作

pom

~~~xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
~~~

application.yml

~~~yaml
spring:
  sleuth:
    web:
      client:
        enabled: true
    sampler:
      probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    base-url: http://localhost:9411/ # 指定了 Zipkin 服务器的地址
~~~

其中spring.sleuth.web.client.enable为true设置的是web开启sleuth功能;spring.sleuth.sampler.probability可以设置为小数，最大值为1.0，当设置为1.0时就是链路数据100%收集到zipkin-server，当设置为0.1时，即10%概率收集链路数据;spring.zipkin.base-url设置zipkin-server的地址。

---

### 四、使用rabbitmq进行链路数据收集

启动zipkin

~~~
java -jar zipkin-server-2.10.1-exec.jar  --zipkin.collector.rabbitmq.addresses=127.0.0.1 --zipkin.collector.rabbitmq.username=root --zipkin.collector.rabbitmq.password=root
~~~

需要eureka-client和eureka-client-feign的起步依赖加上rabbitmq的依赖，依赖如下：

~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
~~~

在配置文件上需要配置rabbitmq的配置，配置信息如下：

~~~yaml
spring: 
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
~~~

另外需要把spring.zipkin.base-url去掉。

| 属性                                         | 环境变量                  | 描述                                                         |
| -------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| zipkin.collector.rabbitmq.addresses          | RABBIT_ADDRESSES          | 用逗号分隔的 RabbitMQ 地址列表，例如localhost:5672,localhost:5673 |
| zipkin.collector.rabbitmq.password           | RABBIT_PASSWORD           | 连接到 RabbitMQ 时使用的密码，默认为 guest                   |
| zipkin.collector.rabbitmq.username           | RABBIT_USER               | 连接到 RabbitMQ 时使用的用户名，默认为guest                  |
| zipkin.collector.rabbitmq.virtual-host       | RABBIT_VIRTUAL_HOST       | 使用的 RabbitMQ virtual host，默认为 /                       |
| zipkin.collector.rabbitmq.use-ssl            | RABBIT_USE_SSL            | 设置为true则用 SSL 的方式与 RabbitMQ 建立链接                |
| zipkin.collector.rabbitmq.concurrency        | RABBIT_CONCURRENCY        | 并发消费者数量，默认为1                                      |
| zipkin.collector.rabbitmq.connection-timeout | RABBIT_CONNECTION_TIMEOUT | 建立连接时的超时时间，默认为 60000毫秒，即 1 分钟            |
| zipkin.collector.rabbitmq.queue              | RABBIT_QUEUE              | 从中获取 span 信息的队列，默认为 zipkin                      |

---

### 五、自定义span元素节点

#### 1、代码（？？？咋玩？？？）

```java
@Autowired
Tracer tracer;

在请求的controller
tracer.currentSpan().tag("name","abcd");
```

---

## 新的篇章

​	上面的例子是将链路数据存在内存中，只要zipkin-server重启之后，之前的链路数据全部查找不到了，zipkin是支持将链路数据存储在mysql、cassandra、elasticsearch中的。

### 一、将链路数据存储在Mysql数据库中

#### 1、新建sql表

~~~sql
CREATE TABLE IF NOT EXISTS zipkin_spans (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL,
  `id` BIGINT NOT NULL,
  `name` VARCHAR(255) NOT NULL,
  `parent_id` BIGINT,
  `debug` BIT(1),
  `start_ts` BIGINT COMMENT 'Span.timestamp(): epoch micros used for endTs query and to implement TTL',
  `duration` BIGINT COMMENT 'Span.duration(): micros used for minDuration and maxDuration query'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_spans ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `id`) COMMENT 'ignore insert on duplicate';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`, `id`) COMMENT 'for joining with zipkin_annotations';
ALTER TABLE zipkin_spans ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTracesByIds';
ALTER TABLE zipkin_spans ADD INDEX(`name`) COMMENT 'for getTraces and getSpanNames';
ALTER TABLE zipkin_spans ADD INDEX(`start_ts`) COMMENT 'for getTraces ordering and range';

CREATE TABLE IF NOT EXISTS zipkin_annotations (
  `trace_id_high` BIGINT NOT NULL DEFAULT 0 COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.trace_id',
  `span_id` BIGINT NOT NULL COMMENT 'coincides with zipkin_spans.id',
  `a_key` VARCHAR(255) NOT NULL COMMENT 'BinaryAnnotation.key or Annotation.value if type == -1',
  `a_value` BLOB COMMENT 'BinaryAnnotation.value(), which must be smaller than 64KB',
  `a_type` INT NOT NULL COMMENT 'BinaryAnnotation.type() or -1 if Annotation',
  `a_timestamp` BIGINT COMMENT 'Used to implement TTL; Annotation.timestamp or zipkin_spans.timestamp',
  `endpoint_ipv4` INT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_ipv6` BINARY(16) COMMENT 'Null when Binary/Annotation.endpoint is null, or no IPv6 address',
  `endpoint_port` SMALLINT COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_service_name` VARCHAR(255) COMMENT 'Null when Binary/Annotation.endpoint is null'
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_annotations ADD UNIQUE KEY(`trace_id_high`, `trace_id`, `span_id`, `a_key`, `a_timestamp`) COMMENT 'Ignore insert on duplicate';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`, `span_id`) COMMENT 'for joining with zipkin_spans';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id_high`, `trace_id`) COMMENT 'for getTraces/ByIds';
ALTER TABLE zipkin_annotations ADD INDEX(`endpoint_service_name`) COMMENT 'for getTraces and getServiceNames';
ALTER TABLE zipkin_annotations ADD INDEX(`a_type`) COMMENT 'for getTraces and autocomplete values';
ALTER TABLE zipkin_annotations ADD INDEX(`a_key`) COMMENT 'for getTraces and autocomplete values';
ALTER TABLE zipkin_annotations ADD INDEX(`trace_id`, `span_id`, `a_key`) COMMENT 'for dependencies job';

CREATE TABLE IF NOT EXISTS zipkin_dependencies (
  `day` DATE NOT NULL,
  `parent` VARCHAR(255) NOT NULL,
  `child` VARCHAR(255) NOT NULL,
  `call_count` BIGINT,
  `error_count` BIGINT
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED CHARACTER SET=utf8 COLLATE utf8_general_ci;

ALTER TABLE zipkin_dependencies ADD UNIQUE KEY(`day`, `parent`, `child`);
~~~

#### 2、zipkin启动命令

~~~
STORAGE_TYPE=mysql MYSQL_HOST=localhost MYSQL_TCP_PORT=3306 MYSQL_USER=root MYSQL_PASS=123456 MYSQL_DB=zipkin java -jar zipkin.jar
~~~

等同于

~~~
java -jar zipkin-server-2.10.1-exec.jar --zipkin.torage.type=mysql --zipkin.torage.mysql.host=10.7.131.14 --zipkin.torage.mysql.port=3306 --zipkin.torage.mysql.username=slup1 --zipkin.torage.mysql.password=GSwDFz6@
~~~

~~~
java -jar zipkin-server-2.10.1-exec.jar --zipkin.torage.type=mysql --zipkin.torage.mysql.host=127.0.0.1 --zipkin.torage.mysql.port=3306 --zipkin.torage.mysql.username=root --zipkin.torage.mysql.password=123456 --zipkin.torage.mysql.db=zipkin 

java -jar zipkin-server-2.10.1-exec.jar --STORAGE_TYPE=mysql --MYSQL_DB=zipkin --MYSQL_USER=root --MYSQL_PASS=123456 --MYSQL_HOST=localhost --MYSQL_TCP_PORT=3306

~~~

#### 3、配置api

| 属性                           | 环境变量              | 描述                                                         |
| ------------------------------ | --------------------- | ------------------------------------------------------------ |
| zipkin.torage.type             | STORAGE_TYPE          | 默认的为mem，即为内存，其他可支持的为cassandra、cassandra3、elasticsearch、mysql |
| zipkin.torage.mysql.host       | MYSQL_HOST            | 数据库的host，默认localhost                                  |
| zipkin.torage.mysql.port       | MYSQL_TCP_PORT        | 数据库的端口，默认3306                                       |
| zipkin.torage.mysql.username   | MYSQL_USER            | 连接数据库的用户名，默认为空                                 |
| zipkin.torage.mysql.password   | MYSQL_PASS            | 连接数据库的密码，默认为空                                   |
| zipkin.torage.mysql.db         | MYSQL_DB              | zipkin使用的数据库名，默认是zipkin                           |
| zipkin.torage.mysql.max-active | MYSQL_MAX_CONNECTIONS | 最大连接数，默认是10                                         |

### 二、将链路数据存在在Elasticsearch中




