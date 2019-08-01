## Consul作为注册中心版

### 一、配置中心

#### 1、pom文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com</groupId>
    <artifactId>consul-config</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <description>Consul Config for Spring Cloud</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- 两种请求方式 Start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <!-- 两种请求方式 End -->

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- 配置热更新start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <!-- 配置热更新start -->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- 链路追踪 Start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <!-- 链路追踪 End -->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>consul-config</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

~~~

#### 2、yml篇

~~~yaml
spring:
  cloud:
    bus:
      enabled: true
      trace:
        enabled: true
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1 #样本采集量，默认为0.1，为了测试这里修改为1，正式环境一般使用默认值。
management:
  endpoints:
    web:
      exposure:
        include: "*"
~~~



~~~yaml
server:
  port: 11200
#eureka:   #注册中心配置
#  instance:
#    prefer-ip-address: true
#    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
#    lease-renewal-interval-in-seconds: 30      # 心跳时间，即服务续约间隔时间（缺省为30s）
#    lease-expiration-duration-in-seconds: 90  # 发呆时间，即服务续约到期时间（缺省为90s）
#  server:
#   hostname: localhost
#   appname: eureka
#   port: 11111
#  client:
#    service-url:
#      defaultZone: @profileActivedefaultZone@
spring:
  application:
    name: consul-config-center
  cloud:
    config:
        server:
          git:
            searchPaths: test30
            password: 12345678
            username: miaopeng
            uri: http://10.7.130.39/config/ConfigCenter.git
            basedir: ./config-repo #指定本地clone仓库路径
            force-pull: true #如果本地仓库拉取配置文件失败，强制拉取远程仓库配置
    consul:
      host: localhost
      port: 8500
      discovery:
        register: true #配置启动是否注册服务
        serviceName: ${spring.application.name}
        healthCheckPath: /actuator/health
        healthCheckInterval: 30s
        tags: ${spring.application.name}
        #instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
        instanceId: ${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
~~~

#### 3、java篇

**application**

~~~java
package com.consulconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class,HibernateJpaAutoConfiguration.class})
@EnableConfigServer
@EnableFeignClients
public class ConsulConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulConfigApplication.class, args);
    }

    /**
     * 通过 LoadBalanced 注解 开启 ribbon 负载均衡
     * @return
     */
    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}

~~~

**controller**

~~~java
package com.consulconfig.controller;

import com.consulconfig.client.ConsulService;
import com.consulconfig.ribbon.RibbonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * @program: finchley
 * @Date: 2019/5/5 14:23
 * @Author: Mr.Yang
 * @Description: 两种请求方式demo
 */
@RestController
public class RequestDemo {

    @Autowired
    ConsulService consulService;

    @Autowired
    RibbonService ribbonService;

    @RequestMapping(value = "/feign")
    public String feign(@RequestParam("name") String name) throws Exception{
        System.out.println("这是一个feign请求");
        return consulService.demo(name);
    }

    @RequestMapping(value = "/ribbon")
    public String ribbon(@RequestParam("name") String name){
        return ribbonService.consulService(name);
    }

}
~~~

**feignService**

~~~java
package com.consulconfig.client;

import com.consulconfig.client.impl.ConsulServiceFallback;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @program: finchley
 * @Date: 2019/5/5 14:09
 * @Author: Mr.Yang
 * @Description: Feign请求别的服务获取数据
 */
@FeignClient(value = "consul-service", fallback = ConsulServiceFallback.class)
public interface ConsulService {
    @RequestMapping(value="/test/{name}")
    String demo(@PathVariable("name") String name) throws Exception;
}

~~~

impl

```java
package com.consulconfig.client.impl;

import com.consulconfig.client.ConsulService;

/**
 * @program: finchley
 * @Date: 2019/5/5 14:12
 * @Author: Mr.Yang
 * @Description:
 */
public class ConsulServiceFallback implements ConsulService {
    @Override
    public String demo(String name) {
        System.out.println("Consul-Service 服务不可用");
        return "Consul-Service 服务不可用";
    }
}
```

---

### 二、服务

#### 1、pom篇

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com</groupId>
    <artifactId>consul-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <description>Consul Service Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>

        <!-- consul作为注册中心 start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- consul作为注册中心 end -->

        <!-- config客户端 start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
        <!-- config客户端 end -->

        <!-- Spring Boot 2.0 之后引入zipkin链路需要的jar包 Start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <!-- Spring Boot 2.0 之后引入zipkin链路需要的jar包 end -->

        <!-- 利用 zxing 解析二维码 Start -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>2.2</version>
        </dependency>
        <!-- 利用qrcode解析二维码 End -->

        <!-- 热加载配置文件 Start -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <!-- 热加载配置文件 End -->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <finalName>consul-service</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 2、yml篇

```yaml
spring:
  profiles:
    active: test30
  application:
    name: consul-service
  cloud:
    config:
      profile: ${spring.profiles.active}
      #要读取的配置文件
      name: slup-common
      discovery:
        #配置中心服务名
        service-id: consul-config-center
        #开启config服务发现
        enabled: true
      #失败快速响应
      fail-fast: true
      retry:
        #重试间隔，默认1000s
        initial-interval: 3000
        #重试次数
        max-attempts: 6
        #最大间隔时间，默认2000ms
        max-interval: 2000
    consul:
      host: localhost
      port: 8500
      discovery:
        register: true #配置启动是否注册服务
        serviceName: ${spring.application.name}
        healthCheckPath: /actuator/health
        healthCheckInterval: 30s
        tags: ${spring.application.name}
        #instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
        instanceId: ${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
    #配置动态更新
    bus:
      enabled: true
      trace:
        enabled: true
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1 #样本采集量，默认为0.1，为了测试这里修改为1，正式环境一般使用默认值。
  #main:
    #allow-bean-definition-overriding: true
server:
  port: 12334
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 3、java篇

**Application**

```java
package com.consulservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@SpringBootApplication
@EnableDiscoveryClient
@RefreshScope
public class ConsulServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulServiceApplication.class, args);
    }

}
```

**Controller**

```java
package com.consulservice.Controller;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @program: finchley
 * @Date: 2019/4/26 11:53
 * @Author: Mr.Yang
 * @Description:
 */
@RestController
public class TestController {
    @RequestMapping(value="/test/{name}")
    private String demo(@PathVariable String name){
        System.out.println("name is: " + name );
        return "hello" + " " + name;
    }
}
```

---

### 三、服务调用

​	1、config调用service

​	http://172.20.10.2:11200/feign?name=feign

​	http://172.20.10.2:11200/ribbon?name=ribbon

​	2、service服务自调用

​	http://127.0.0.1:12334/test/1

---

### 四、配置文件动态刷新（神坑！webhook还需要安装配置）

​	例如：

​	1、修改了service服务在配置中心的yml，直接请求 http://127.0.0.1:12334/actuator/bus-refresh （POST）即可热加载。

​	2、当使用git作为配置中心数据文件服务器时，配置如下：

​	配置git的webhook ,当git端配置发生改变，自动调用/bus-refresh接口

![img](https://img-blog.csdnimg.cn/20181104224113454.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d0ZG1fMTYwNjA0,size_16,color_FFFFFF,t_70)



---

### 五、注意事项

​	使用RabbitMQ时，本地环境不需要配置mq，但是需要启动mq，Springboot会自动连接本地mq，后面客户端也是，如果是线上环境的话，必须要进行配置,原因看看SpringCloud bus如下说明：

![img](https://img-blog.csdnimg.cn/20181104225008558.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d0ZG1fMTYwNjA0,size_16,color_FFFFFF,t_70)

---

## 附录

### 1. 加解密

​	配置文件统一放在配置中心，配置中心文件明文存在不安全，容易泄露比如数据库用户名、密码等，如何实现git仓库配置文件为密文时，通过配置中心在Config-Server端进行解密。

#### <一>对称加密

1、JCE加密，Oracl官网下载，替换本机JDK下JRE的lib下在两个文件。

​	https://www.oracle.com/technetwork/java/javase/downloads/jce-all-download-5170447.html 

2、

​	**a) Config-Server 端配置文件添加：**

~~~yaml
encrypt:                                 #加密因子
  key: foobar
~~~


​	加密因子为foobar，这里借助了Server端的加密，因此配置完毕需要启动Config-Server

​	**b) 启动Config-Server**

~~~
$ curl -X post http://localhost:9090/encrypt -d mysecret
~~~

​	加密mysecret为密码，得到如下加密字串：

~~~
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
~~~

​	**c) 逆向操作：**

~~~
$ curl -X post http://localhost:9090/decrypt -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
~~~

​	解密得到：

~~~
mysecret
~~~

3、使用：

​	**a) Config-Server端配置文件中加入：**

~~~yaml
encrypt:                                 #加密因子foobar
  key: foobar
~~~

​	**b) Git仓库中配置文件caiyun-test-dev.yml**

~~~
profile: '{cipher}682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda'
~~~

​	备注：.properties文件吧引号去掉

​	**c) Config-client不需要做任何操作**

#### <二>非对称加密

**RSA算法**

1、命令行下执行

~~~
$ keytool -genkeypair -alias mytestkey -keyalg RSA -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" -keypass changeme -keystore server.jks -storepass letmein
~~~

​	执行完生成 server.jks 文件，加密文件

2、将Server.jks 文件放在Config-Server的ClassPass路径下，

​	Config-Server 端配置文件bootstrap.yml中添加：

~~~yaml
encrypt:
  keyStore:
    location: classpath:/server.jks		#生成在jks文件路径
    password: letmein			#key store 秘钥
    alias: mytestkey			#别名
    secret: changeme			#私钥
~~~

​	启动Config-Server

3、加密

~~~
$ curl -X post http://localhost:9090/encrypt -d caiyun-mima
~~~

​	加密caiyun-mima，得到如下加密字串：

~~~
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
~~~

4、使用：

​	a）Config-Server端配置文件中加入：

~~~yaml
encrypt:
  keyStore:
    location: classpath:/server.jks		#生成在jks文件路径
    password: letmein			#key store 秘钥
    alias: mytestkey			#别名
    secret: changeme			#私钥	
~~~


​	b) Git仓库中配置文件caiyun-test-dev.yml

~~~
profile: '{cipher}682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda'
~~~

​	备注：.properties文件吧引号去掉

​	c) Config-client不需要做任何操作

​	