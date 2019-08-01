## 	Eureka

### 一、region和zone的使用

#### 1、需求背景

​	用户量比较大或者用户地理位置分布范围很广的项目，一般都会有多个机房。这个时候如果上线springCloud服务的话，我们希望一个机房内的服务优先调用同一个机房内的服务，当同一个机房的服务不可用的时候，再去调用其它机房的服务，以达到减少延时的作用。

#### 2、概念

eureka提供了region和zone两个概念来进行分区，这两个概念均来自于亚马逊的AWS：

​	（1）region：可以简单理解为地理上的分区，比如亚洲地区，或者华北地区，再或者北京等等，没有具体大小的限制。根据项目具体的情况，可以自行合理划分region。

​	（2）zone：可以简单理解为region内的具体机房，比如说region划分为北京，然后北京有两个机房，就可以在此region之下划分出zone1,zone2两个zone。

#### 3、部署示例

![img](D:\Typora文档\文档\学习文档\Spring Cloud\Eureka-Zone-region.png)

​	如上图所示，有一个region:beijing，下面有zone-1和zone-2两个分区，每个分区内有一个注册中心Eureka Server和一个服务提供者Service。我们在zone-1内创建一个

Consumer-1服务消费者的话，其会优先调用同一个zone内的Service-1，当Service-1不可用时，才会去调用zone-2内的Service-2。

#### 4、实战

**eureka端配置新增：**

~~~yaml
eureka:
  client:
    prefer-same-zone-eureka: true
    #地区
    region: beijing
    availability-zones:
      beijing: zone-1,zone-2
    service-url:
      zone-1: http://localhost:30000/eureka/
      zone-2: http://localhost:30001/eureka/
~~~

​	**说明**：如果prefer-same-zone-eureka为true，先通过region取availability-zones内的第一个zone，然后通过这个zone取service-url下的list，并向list内的第一个注册中心进行注册和维持心跳，不会再向list内的其它的注册中心注册和维持心跳。只有在第一个注册失败的情况下，才会依次向其它的注册中心注册，总共重试3次，如果3个service-url都没有注册成功，则注册失败。每隔一个心跳时间，会再次尝试。

​	如果prefer-same-zone-eureka为false，按照service-url下的 list取第一个注册中心来注册，并和其维持心跳检测。不会再向list内的其它的注册中心注册和维持心跳。只有在第一个注册失败的情况下，才会依次向其它的注册中心注册，总共重试3次，如果3个service-url都没有注册成功，则注册失败。每隔一个心跳时间，会再次尝试。

​	此处需注意：为了保证服务注册到同一个zone的注册中心，在上述配置为true时，一定要注意availability-zones的顺序，必须把同一zone写在前面。

**client端配置新增**

~~~yaml
eureka:
  instance:
    metadata-map:
      zone: zone-1
  client:
    prefer-same-zone-eureka: true
    #地区
    region: beijing
    availability-zones:
      beijing: zone-1,zone-2
    service-url:
      zone-1: http://localhost:30000/eureka/
      zone-2: http://localhost:30001/eureka/
zone.name: zone-1
~~~

​	**注意：**

​	（1）服务注册：要保证服务注册到同一个zone内的注册中心，因为如果注册到别zone的注册中心的话，网络延时比较大，心跳检测很可能出问题。

​	（2）服务调用：要保证优先调用同一个zone内的服务，只有在同一个zone内的服务不可用时，才去调用别zone的服务。

​	（3）服务消费者和服务提供者分别属于哪个zone，均是通过eureka.instance.metadata-map.zone来判定的。服务消费者会先通过ribbon去注册中心拉取一份服务提供者的列表，然后通过eureka.instance.metadata-map.zone指定的zone进行过滤，过滤之后如果同一个zone内的服务提供者有多个实例，则会轮流调用。只有在同一个zone内的所有服务提供者都不可用时，才会调用其它zone内的服务提供者。