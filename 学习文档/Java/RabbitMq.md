## windows下安装rabbitMq及操作常用命令

​	rabbitMQ是一个在AMQP协议标准基础上完整的，可服用的企业消息系统。它遵循Mozilla Public License开源协议，采用 Erlang 实现的工业级的消息队列(MQ)服务器，Rabbit MQ 是建立在Erlang OTP平台上。

​	默认端口：5672

​	http查看端口：15672

### 一、安装Erlang

​	在安装rabbitMQ之前，需要先安装Erlang。

​	官网地址：http://www.erlang.org/downloads

​		注：这个官网很坑，基本一访问就挂，建议翻墙或者把下载地址直接放在迅雷中下载。

​	或者使用解压包安装，解压安装包后运行**vcredist_x64.exe**文件即可（此处留坑，本人暂时未解决）

​	安装完成后，查看环境变量，如下图

​	![环境变量图片](https://images2015.cnblogs.com/blog/784082/201609/784082-20160923235447637-1807926011.png)	

### 二、安装RabbitMQ

​	官网下载地址：https://www.rabbitmq.com/install-windows.html

​	注意事项：**安装时必须以管理员省份运行，需要到注册表权限，默认安装的RabbitMQ 监听端口是5672**

#### 	1、安装完成后运行如下命令：

~~~java
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmq-plugins.bat enable rabbitmq_management
~~~

​	**命令行得到如下提示即可：**

~~~java
Enabling plugins on node rabbit@DESKTOP-9RFI5FU:
rabbitmq_management
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@DESKTOP-9RFI5FU...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

started 3 plugins.
~~~

---

#### 	2、重启服务才行，使用命令：

~~~
net stop RabbitMQ && net start RabbitMQ
~~~

​	若得到如下，表示当前不是管理员运行CMD

~~~java
发生系统错误 5。

拒绝访问。
~~~

​	**切换为管理员状态运行CMD**

​	**运行命令时若识别不了 && 符号，请把两条语句分开来操作。**

~~~java
PS D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin> net stop RabbitMQ
RabbitMQ 服务正在停止......
RabbitMQ 服务已成功停止。

PS D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin> net start RabbitMQ
RabbitMQ 服务正在启动 .
RabbitMQ 服务已经启动成功。

rabbitmqctl.bat status 查看启停状态
~~~

---

#### 3、创建用户，密码，绑定角色

​	使用rabbitmqctl控制台命令（位于 .\RabbitMQ Server\rabbitmq_server-3.6.5\sbin>）来创建用户，密码，绑定权限等。

​	**注意：安装路径不同的请看仔细啊。需要切换到CMD窗口运行下**

​	rabbitmq的**用户管理**包括**增加用户，删除用户，查看用户列表，修改用户密码**。

---

​	**查看已有用户及用户的角色：**

~~~
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmqctl.bat list_users
Listing users ...
user    tags
guest   [administrator]
~~~

​	注释：

​	rabbitmq用户角色可分为五类：超级管理员, 监控者, 策略制定者, 普通管理者以及其他。

​	(1) 超级管理员(administrator)

​	可登陆管理控制台(启用management plugin的情况下)，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

​	(2) 监控者(monitoring)

​	可登陆管理控制台(启用management plugin的情况下)，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等) 

​	(3) 策略制定者(policymaker)

​	可登陆管理控制台(启用management plugin的情况下), 同时可以对policy进行管理。

​	(4) 普通管理者(management)

​	仅可登陆管理控制台(启用management plugin的情况下)，无法看到节点信息，也无法对策略进行管理。

​	(5) 其他的

​	无法登陆管理控制台，通常就是普通的生产者和消费者。

---

​	**新增一个用户**

​	命令格式：rabbitmqctl.bat add_user user password

~~~
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmqctl.bat add_user root root
Adding user "root" ...
~~~

---

​	**分配角色**

​	命令格式：rabbitmqctl.bat set_user_tags username role1 role2 ...

```
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmqctl.bat set_user_tags root administrator
Setting tags for user "root" to [administrator] ...
```

---

​	**更改密码**

​	命令格式：rabbitmqctl change_password userName newPassword

~~~
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmqctl change_password guest 123456
Changing password for user "guest" ...
~~~

---

​	**删除用户**

​	命令格式：rabbitmqctl.bat delete_user username

~~~
D:\RabbitMq-3.7.14\rabbitmq_server-3.7.14\sbin>rabbitmqctl.bat delete_user guest
Deleting user "guest" ...
~~~

---

### 三、页面操作

​	页面登陆地址：http://localhost:15672/

​	用户名密码：root/root （之前设置的）

---

### 四、权限设置

​	按照官方文档，用户权限指的是用户对**exchange，queue**的操作权限，包括配置权限，读写权限。

​	我们配置权限会影响到exchange、queue的声明和删除。

​	读写权限影响到从queue里取消息、向exchange发送消息以及queue和exchange的绑定(binding)操作。

​	例如： 将queue绑定到某exchange上，需要具有queue的可写权限，以及exchange的可读权限；向exchange发送消息需要具有exchange的可写权限；从queue里取数据需要具有queue的可读权限。 

​	权限相关命令为：

​	**(1) 设置用户权限**

​	rabbitmqctl  set_permissions  -p  VHostPath  User  ConfP  WriteP  ReadP

​	**(2) 查看(指定hostpath)所有用户的权限信息**

​	rabbitmqctl  list_permissions  [-p  VHostPath]

​	**(3) 查看指定用户的权限信息**

​	rabbitmqctl  list_user_permissions  User

​	**(4)  清除用户的权限信息**

​	rabbitmqctl  clear_permissions  [-p VHostPath]  User

---

## 参考：

安装参考 <http://www.rabbitmq.com/install-windows-manual.html>

权限内容参考 <http://www.rabbitmq.com/man/rabbitmqctl.1.man.html>

权限命令摘自 <https://my.oschina.net/hncscwc/blog/262246>

---

## 附录：

### 1、安装程序:

[RabbitMQ安装程序](D:\Typora文档\文档\学习文档\Java\rabbitMq.rar)

### 2、结合Spring boot 时

#### 1.java.net.SocketException: socket closed

​	官方文档已经说明，新建user和guest的账户是没有远程登录的权限的 需要对登录所用账户授权

　　　解决方法：

```
rabbitmqctl  set_permissions -p /${user_name}  user_admin '.*' '.*' '.*'
```

---

#### 2.An unexpected connection driver error occured  报错如下

~~~java
[AMQP Connection 192.168.71.111:5672] WARN com.rabbitmq.client.impl.ForgivingExceptionHandler - An unexpected connection driver error occured (Exception message: Connection reset)
Exception in thread "main" com.rabbitmq.client.AuthenticationFailureException: ACCESS_REFUSED - Login was refused using authentication mechanism PLAIN. For details see the broker logfile.
    at com.rabbitmq.client.impl.AMQConnection.start(AMQConnection.java:342)
    at com.rabbitmq.client.impl.recovery.RecoveryAwareAMQConnectionFactory.newConnection(RecoveryAwareAMQConnectionFactory.java:62)
    at com.rabbitmq.client.impl.recovery.AutorecoveringConnection.init(AutorecoveringConnection.java:99)
    at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:900)
    at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:859)
    at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:817)
    at com.rabbitmq.client.ConnectionFactory.newConnection(ConnectionFactory.java:954
~~~

这是因为Rabbit里的queue 没有 注解@RabbitListener(queues = "namedQueue")  queues的实例

　　　　解决方法：![技术分享图片](http://image.mamicode.com/info/201806/20180629191634973488.png)

　　添加监听的queues名称，保存  重启 。OK