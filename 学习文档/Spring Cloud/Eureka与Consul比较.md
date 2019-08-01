## 写在前面

​	为什么要拿Consul和Eureka进行比较呢？是因为 自2018-06开始 Eureka2.X版本已经宣布闭源，即Eureka 2.X 遇到困难停止开发了。

## Consul

​	[Consul网页地址](https://blog.csdn.net/forezp/article/details/87273153)

### 1、背景关系篇：

HashiCorp公司、Go语言。

提供了**分布式系统的服务注册和发现、配置**等功能，这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格。

---

### 2、优点：

Consul不仅具有服务治理的功能，而且使用分布式一致协议RAFT算法实现，有多数据中心的高可用方案，并且很容易和Spring Cloud等微服务框架集成，使用起来非常的简单，具有简单、易用、可插排等特点。使用简而言之，Consul提供了一种完整的服务网格解决方案 。

---

### 3、Consul具有以下的特点和功能

#### （1）服务发现：

​	Consul的客户端可以向Consul注册服务，例如api服务或者mysql服务，其他客户端可以使用Consul来发现服务的提供者。Consul支持使用DNS或HTTP来注册和发现服务。

#### （2）运行时健康检查：

​	Consul客户端可以提供任意数量的运行状况检查机制，这些检查机制可以是给定服务（“是Web服务器返回200 OK”）或本地节点（“内存利用率低于90％”）相关联。这些信息可以用来监控群集的运行状况，服务发现组件可以使用这些监控信息来路由流量，可以使流量远离不健康的服务。

#### （3）KV存储：

​	应用程序可以将Consul的键/值存储用于任何需求，包括动态配置，功能标记，协调，领导者选举等。它采用HTTP API使其易于使用。

#### （4）安全服务通信：

​	Consul可以为服务生成和分发TLS证书，以建立相互的TLS连接。

#### （5）多数据中心：

​	Consul支持多个数据中心。这意味着Consul的用户不必担心构建额外的抽象层以扩展到多个区域。

​	内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。

---

### 4、Consul原理

（看看就行了，用的多了自然就知道了）

​	每个提供服务的节点都运行了Consul的代理，运行代理不需要服务发现和获取配置的KV键值对，代理只负责监控检查。代理节点可以和一个或者多个Consul server通讯。 Consul服务器是存储和复制数据的地方。服务器本身选出了领导者。虽然Consul可以在一台服务器上运行，但建议使用3到5，以避免导致数据丢失的故障情况。建议为每个数据中心使用一组Consul服务器。
如果你的组件需要发现服务，可以查询任何Consul Server或任何Consul客户端，Consul客户端会自动将查询转发给Consul Server。

​	需要发现其他服务或节点的基础架构组件可以查询任何Consul服务器或任何Consul代理。代理会自动将查询转发给服务器。每个数据中心都运行Consul服务器集群。发生跨数据中心服务发现或配置请求时，本地Consul服务器会将请求转发到远程数据中心并返回结果。

---

### 5、Consul PK Eureka

#### (1)、CAP架构比较

​	Consul 使用的是CP架构（强一致性），Eureka使用的是AP架构（高可用性）

​	注：CAP架构详见 [CAP架构文档](D:\Typora文档\文档\学习文档\Spring Cloud\CAP理论.md)

##### 	Consul强一致性(C)带来的是：

​		服务注册相比Eureka会稍慢一些。因为Consul的raft协议要求必须过半数的节点都写入成功才认为注册成功Leader挂掉时，重新选举期间整个consul不可用。保证了强一致性但牺牲了可用性。

##### 	Eureka保证高可用(A)和最终一致性：

​	服务注册相对要快，因为不需要等注册信息replicate到其他节点，也不保证注册信息是否replicate成功

​	当数据出现不一致时，虽然A, B上的注册信息不完全相同，但每个Eureka节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到。如此保证了可用性但牺牲了一致性。

| 比较项/服务          | **Consul**             | **zookeeper**         | **etcd**          | **euerka**                   |
| -------------------- | ---------------------- | --------------------- | ----------------- | ---------------------------- |
| 服务健康检查         | 服务状态，内存，硬盘等 | (弱)长连接，keepalive | 连接心跳          | 可配支持                     |
| 多数据中心           | 支持                   | —                     | —                 | —                            |
| kv存储服务           | 支持                   | 支持                  | 支持              | —                            |
| 一致性               | raft                   | paxos                 | raft              | —                            |
| CAP架构              | cp                     | cp                    | cp                | ap                           |
| 使用接口(多语言能力) | 支持http和dns          | 客户端                | http/grpc         | http（sidecar）              |
| watch支持            | 全量/支持long polling  | 支持                  | 支持 long polling | 支持 long polling/大部分增量 |
| 自身监控             | metrics                | —                     | metrics           | metrics                      |
| 安全                 | acl /https             | acl                   | https支持（弱）   | —                            |
| spring cloud集成     | 已支持                 | 已支持                | 已支持            | 已支持                       |

- **服务的健康检查**

  Euraka 使用时需要显式配置健康检查支持；Zookeeper,Etcd 则在失去了和服务进程的连接情况下任务不健康，而 Consul 相对更为详细点，比如内存是否已使用了90%，文件系统的空间是不是快不足了。

- **多数据中心支持**

  Consul 通过 WAN 的 Gossip 协议，完成跨数据中心的同步；而且其他的产品则需要额外的开发工作来实现；

  注：WAN Gossip 它只包含Server。这些server主要分布在不同的数据中心并且通常通过因特网或者广域网通信。

- **KV 存储服务**

  除了 Eureka ,其他几款都能够对外支持 k-v 的存储服务，所以后面会讲到这几款产品追求高一致性的重要原因。而提供存储服务，也能够较好的转化为动态配置服务哦。

- **多语言能力与对外提供服务的接入协议**

  Zookeeper的跨语言支持较弱，其他几款支持 http11 提供接入的可能。Euraka 一般通过 sidecar的方式提供多语言客户端的接入支持。Etcd 还提供了Grpc的支持。 Consul除了标准的Rest服务api,还提供了DNS的支持。

- **Watch的支持（客户端观察到服务提供者变化）**

  Zookeeper 支持服务器端推送变化，Eureka 2.0(正在开发中)也计划支持。 Eureka 1,Consul,Etcd则都通过长轮询的方式来实现变化的感知；

- **自身集群的监控**

  除了 Zookeeper ,其他几款都默认支持 metrics，运维者可以搜集并报警这些度量信息达到监控目的；

- **安全**

  Consul,Zookeeper 支持ACL，另外 Consul,Etcd 支持安全通道https.

- **Spring Cloud的集成**

  目前都有相对应的 boot starter，提供了集成能力。

#### (2)、开源性

​	Eureka是在Java语言上，基于Restful Api开发的服务注册与发现组件，由Netflix开源。遗憾的是，目前Eureka仅开源到1.X版本，2.X版本已经宣布闭源。

#### (3)、调用方式

​	Consul采用主从模式的设计，使得集群的数量可以大规模扩展，集群间通过RPC的方式调用(HTTP和DNS)。

​	Eureka采用Restf API调用方式。

#### (4)、服务治理

​	**Spring Cloud Eureka**

​	Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

​	**Spring Cloud Consul**

​	Spring Cloud Consul项目是针对Consul的服务治理实现。Consul是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。它包含了下面几个特性：

 + **服务发现**

 + **健康检查**

 + **Key/Value存储**

 + **多数据中心**

   由于Spring Cloud Consul项目的实现，我们可以轻松的将基于Spring Boot的微服务应用注册到Consul上，并通过此实现微服务架构中的服务治理。

---

## 附录：

### 1、Raft算法

​	Raft算法将Server分为三种类型：Leader、Follower和Candidate。Leader处理所有的查询和事务，并向Follower同步事务。Follower会将所有的RPC查询和事务转发给Leader处理，它仅从Leader接受事务的同步。数据的一致性以Leader中的数据为准实现。

​	在节点初始启动时，节点的Raft状态机将处于Follower状态等待来来自Leader节点的心跳。如果在一定时间周期内没有收到Leader节点的心跳，节点将发起选举。

​	Follower节点选举时会将自己的状态切换为Candidate，然后向集群中其它Follower节点发送请求，询问其是否选举自己成为Leader。当收到来自集群中过半数节点的接受投票后，节点即成为Leader，开始接收Client的事务处理和查询并向其它的Follower节点同步事务。Leader节点会定时向Follower发送心跳来保持其地位。

### 2、Gossip协议

​	Gossip协议是为了解决分布式环境下监控和事件通知的瓶颈。Gossip协议中的每个Agent会利用Gossip协议互相检查在线状态，分担了服务器节点的心跳压力，通过Gossip广播的方式发送消息。

​	所有的Agent都运行着Gossip协议。服务器节点和普通Agent都会加入这个Gossip集群，收发Gossip消息。每隔一段时间，每个节点都会随机选择几个节点发送Gossip消息，其他节点会再次随机选择其他几个节点接力发送消息。这样一段时间过后，整个集群都能收到这条消息。

​	基于Raft算法，Consul提供强一致性的注册中心服务，但是由于Leader节点承担了所有的处理工作，势必加大了注册和发现的代价，降低了服务的可用性。通过Gossip协议，Consul可以很好地监控Consul集群的运行，同时可以方便通知各类事件，如Leader选择发生、Server地址变更等。

