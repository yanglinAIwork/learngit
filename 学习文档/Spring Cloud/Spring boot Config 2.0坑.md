## Eureka作为注册中心版

### 一、配置文件在Git时

#### 1、pom配置

​	此处的坑：spring-cloud-starter-eureka-client 包变为 spring-cloud-starter-netflix-eureka-client

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
    <artifactId>eureka-config</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-config</name>
    <description>Eureka Config for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

~~~

#### 2、yml配置

​	此处的坑：之前的spring.cloud.client.ipAddress改为spring.cloud.client.ip-address ，否则取不到ip地址；

​			   basedir由之前的路径名（config-repo3）改为相对路径（./config-repo3）或者绝对路径（C:\\）

```yaml
server:
  port: 11200

eureka:   #注册中心配置
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${spring.cloud.client.ip-address}:${spring.application.instance_id:${server.port}}
    lease-renewal-interval-in-seconds: 30      # 心跳时间，即服务续约间隔时间（缺省为30s）
    lease-expiration-duration-in-seconds: 90  # 发呆时间，即服务续约到期时间（缺省为90s）
  client:
    service-url:
      defaultZone: http://127.0.0.1:11111/eureka

spring:
  application:
    name: eureka-config-server
  cloud:
    config:
      server:
        git:
          searchPaths: demo
          password: password
          username: username
          uri: http://10.7.130.39/config/ConfigCenter.git
          basedir: ./config-repo3 #指定本地clone仓库路径
```

#### 3、启动类

~~~java
@EnableConfigServer
~~~



---

### 二、配置文件在数据库时

暂未收集

---

## Consul作为注册中心版