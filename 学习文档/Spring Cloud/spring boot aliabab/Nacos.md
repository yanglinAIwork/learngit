## Nacos

​	官方网站：https://nacos.io/zh-cn/

### 一、环境准备

1. 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
2. 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) & [配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
3. Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi) & [配置](https://maven.apache.org/settings.html)。

### 二、下载源码或者安装包<本地没下载，直接在docker跑的。>

​	1、下载Nacos包

​	2、cd nacos/bin

### 三、启动服务(单机版)

​	**Linux/Unix/Mac**

​	启动命令(standalone代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone
```

​	**Windows**

​	启动命令：

```
cmd startup.cmd-m standalone
```

​	或者双击startup.cmd运行文件。

### 四、服务注册&发现和配置管理

#### 1、服务注册

```
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
```

#### 2、服务发现

```
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instances?serviceName=nacos.naming.serviceName'
```

#### 3、发布配置

```
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"
```

#### 4、获取配置

```
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
```

### 五、关闭服务器

​	**Linux/Unix/Mac**

```
sh shutdown.sh
```

​	**Windows**

```
cmd shutdown.cmd
```

​	或者双击shutdown.cmd运行文件。

---

## 附录

### 1、eureka1.0版本缺陷

+ 阅端拿到的是服务的全量的地址：这个对于客户端的内存是一个比较大的消耗，特别在多数据中心部署的情况下，某个数据中心的订阅端往往只需要同数据中心的服务提供端即可。
+ pull 模式：客户端采用周期性向服务端主动 pull 服务数据的模式（也就是客户端轮训的方式），这个方式存在实时性不足以及无谓的拉取性能消耗的问题。
+ 一致性协议：Eureka 集群的多副本的一致性协议采用类似“异步多写”的 AP 协议，每一个 server 都会把本地接收到的写请求（register/heartbeat/unregister/update）发送给组成集群的其他所有的机器（Eureka 称之为 peer），特别是 hearbeat 报文是周期性持续不断的在 client->server->all peers 之间传送；这样的一致性算法，导致了如下问题
  +  每一台 Server 都需要存储全量的服务数据，Server 的内存明显会成为瓶颈。
  + 当订阅者却来越多的时候，需要扩容 Eureka 集群来提高读的能力，但是扩容的同时会导致每台 server 需要承担更多的写请求，扩容的效果不明显。
  + 组成 Eureka 集群的所有 server 都需要采用相同的物理配置，并且只能通过不断的提高配置来容纳更多的服务数据

### 2、Eureka2.0 改进和增强

 * 数据推送从 pull 走向 push 模式，并且实现更小粒度的服务地址按需订阅的功能。

 * 读写分离：写集群相对稳定，无需经常扩容；读集群可以按需扩容以提高数据推送能力。

   但是 TMD 闭源了，你能忍？阿里也不能忍，就搞出个spring boot alibaba 这玩意来。

### 3、与eureka区别：

​	支持 RESTful 、RPC 调用协议。

​	没闭源。



