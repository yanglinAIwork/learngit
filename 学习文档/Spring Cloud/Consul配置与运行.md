## 一、Consul下载和安装

​	Consul采用Go语言编写，支持Linux、Mac、Windows等各大操作系统，本文使用windows操作系统，下载地址：https://www.consul.io/downloads.html，下完成后解压到计算机目录下，解压成功后，只有一个可执行的consul.exe可执行文件。打开cmd终端，切换到目录，执行以下命令：

~~~markdown
consul --version
~~~

终端显示如下：

```
Consul v1.4.2
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use p
rotocol >2 when speaking to compatible agents)
```

证明consul下载成功了，并可执行。

consul的一些常见的执行命令如下：

|  命令   |               解释                |       示例        |
| :-----: | :-------------------------------: | :---------------: |
|  agent  |       运行一个consul agent        | consul agent -dev |
|  join   |      将agent加入到consul集群      |  consul join IP   |
| members | 列出consul cluster集群中的members |  consul members   |
|  leave  |        将节点移除所在集群         |   consul leave    |

**更多命令请查看官方网站**：<https://www.consul.io/docs/commands/index.html>

开发模式启动：

```
consul agent -dev 
```


启动成功，在浏览器上访问：http://localhost:8500，显示的界面如下：

![Consul主界面](https://upload-images.jianshu.io/upload_images/2279594-087c91873bfa1b67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

## 二、spring cloud consul

​	该项目通过自动配置并绑定到Spring环境和其他Spring编程模型成语，为Spring Boot应用程序提供Consul集成。通过几个简单的注释，您可以快速启用和配置应用程序中的常见模式，并使用基于Consul的组件构建大型分布式系统。提供的模式包括服务发现，控制总线和配置。智能路由（Zuul）和客户端负载平衡（Ribbon），断路器（Hystrix）通过与Spring Cloud Netflix的集成提供。

### 1、使用spring cloud consul来服务注册与发现

​	本小节以案例的形式来讲解如何使用Spring Cloud Consul来进行服务注册和发现的，并且使用Feign来消费服务。再讲解之前，已经启动consul的agent，并且在浏览器上http://localhost:8500能够显示正确的页面。本案例一共有2个工程，分别如下：

|     工程名      | 端口 |    描述    |
| :-------------: | :--: | :--------: |
| consul-provider | 8763 | 服务提供者 |
| consul-consumer | 8765 | 服务消费者 |

**服务提供者consul-provider**

​	创建一个工程consul-provider，在工程的pom文件引入以下依赖，包括consul-discovery的起步依赖，该依赖是spring cloud consul用来向consul 注册和发现服务的依赖，采用REST API的方式进行通讯。另外加上web的起步依赖，用于对外提供REST API。代码如下：

```java
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

​	在工程的配置文件application.yml做下以下配置：

```java
server:
  port: 8763
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-provider
```

​	在程序员的启动类ConsulProviderApplication加上@EnableDiscoveryClient注解，开启服务发现的功能。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsulProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsulProviderApplication.class, args);
	}
}
```

​	再写一个可调用的方法**RESTAPI**，启动服务。

**服务消费者consul-provider**

​	写一个FeignClient，该FeignClient调用consul-provider的REST API，代码如下：

```java
@FeignClient(value = "consul-provider")
public interface EurekaClientFeign {

 
    @GetMapping(value = "/hi")
    String sayHiFromClientEureka(@RequestParam(value = "name") String name);
}
```

​	Service层代码如下：

```java
@Service
public class HiService {

    @Autowired
    EurekaClientFeign eurekaClientFeign;
 
   
    public String sayHi(String name){
        return  eurekaClientFeign.sayHiFromClientEureka(name);
    }
}
```

​	对外提供一个REST API，该API调用了consul-provider的服务，代码如下：

```java
@RestController
public class HiController {
    @Autowired
    HiService hiService;

    @GetMapping("/hi")
    public String sayHi(@RequestParam( defaultValue = "forezp",required = false)String name){
        return hiService.sayHi(name);
    }
}
```

### 2、使用Spring Cloud Consul Config来做服务配置中心

​	实践地址：https://my.oschina.net/giegie/blog/1634939

​	Consul不仅能用来服务注册和发现，Consul而且支持Key/Value键值对的存储，可以用来做配置中心。Spring Cloud 提供了Spring Cloud Consul Config依赖去和Consul相集成，用来做配置中心。

​	现在以案例的形式来讲解如何使用Consul作为配置中心，本案例在上一个案例的consul-provider基础上进行改造。首先在工程的pom文件加上consul-config的起步依赖，代码如下：

~~~java
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
~~~

​	然后在配置文件application.yml加上以下的以下的配置，配置如下：

```java
spring:
  profiles:
    active: dev 
```

​	上面的配置指定了SpringBoot启动时的读取的profiles为dev。然后再工程的启动配置文件bootstrap.yml文件中配置以下的配置：

~~~java
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-provider
      config:
        enabled: true
        format: yaml           
        prefix: config     
        profile-separator: ':'    
        data-key: data    
~~~

关于spring.cloud.consul.config的配置项描述如下：

+ **enabled 设置config是否启用，默认为true**
+ **format 设置配置的值的格式，可以yaml和properties**
+ **prefix 设置配置的基本目录，比如config**
+ **defaultContext 设置默认的配置，被所有的应用读取，本例子没用的**
+ **profileSeparator profiles配置分隔符,默认为‘,’**
+ **date-key为应用配置的key名字，值为整个应用配置的字符串。**

---

网页上访问consul的KV存储的管理界面，即http://localhost:8500/ui/dc1/kv，创建一条记录，

key值为：config/consul-provider:dev/data
value值如下:

~~~java
foo:
  bar: bar1
server:
  port: 8081
~~~

![Consul配置中心地址](https://upload-images.jianshu.io/upload_images/2279594-9064649b16ac4bf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​	在consul-provider工程新建一个API，该API返回从consul 配置中心读取foo.bar的值，代码如下：

~~~java
@RestController
public class FooBarController {
    
    @Value("${foo.bar}")
    String fooBar;

    @GetMapping("/foo")
    public String getFooBar() {
        return fooBar;
    }
}
~~~

### 3、动态刷新配置

​	当使用spring cloud config作为配置中心的时候，可以使用spring cloud config bus支持动态刷新配置。Spring Cloud Comsul Config默认就支持动态刷新，只需要在需要动态刷新的类上加上**@RefreshScope**注解即可。

## 三、注意事项

- consul支持的KV存储的Value值不能超过512KB
- Consul的dev模式，所有数据都存储在内存中，重启Consul的时候会导致所有数据丢失，在正式的环境中，Consul的数据会持久化，数据不会丢失。
- 暂时未使用consul做配置中心，因为consul远程Git做配置中心时需要一个Git2Consul插件。（虽然和cloud的config相比不需要引入bus就能动态刷新，但是使用插件的话总感觉不可控）

---

## 附录：

### 1、停止consul无效服务

~~~
PUT http://127.0.0.1:8500/v1/agent/service/deregister/consul-config-center-172-20-10-2-11200

consul-config-center-172-20-10-2-11200 后边拼的是serviceId，服务名称+ip
~~~

### 2、停止consul无效节点

~~~
http://server_ip:8500/v1/agent/force-leave/4b36b27317a0
~~~



### 

