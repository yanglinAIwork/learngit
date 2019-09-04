## AOP

### 一、源码篇

---

### 二、应用级别

​		场景：权限验证、日志记录、限流

​					aop分库分表、aop如何做分布式事务

#### 1、AOP如何做分库分表： 未完待续

​		分库分表：主要分担生产写库压力<32个生产写库>，三个策略

​				多库多表（4个订单库，每个库的表结构有N个，order_001表、order_002表...）每个表存m条，可存放4\*N\*m条数据	制定一个实体类：`RoutingDsAndTbStategy`

​				多库一表（4个订单库，每个库只有一个表）	制定一个实体类：`RoutingDsStategy`

​				一库多表（1个订单库，有多个表）	制定一个实体类：`RoutingTbStategy`



做一个接口`ITulingRouting`定义行为：

​	1、计算数据具体存入到哪个库的行为

​	2、计算选择表的行为

​	3、计算选择表后缀的行为



如果三个实体类都要实现这个接口，那么引入一个抽象类`AbstractTulingRouting`：

**<此时用到了设计模式：模板策略>**



a、接口类的方法

![1567081518844](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接口类.png)

b、把接口类的公共方法提取出来放在抽象类中

​	抽象类的代码

![1567081880169](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-抽象类1.png)

![1567081912780](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-抽象类2.png)

![1567083133926](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-抽象类3.png)

![1567083215127](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-抽象类4.png)

![1567083244578](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-抽象类5.png)

分库分表的配置

![1567081980449](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-yml配置.png)

​		第一个数据库的个数，第二个数据库表的个数，第三个是指定入参中路由字段，第四、五个是指定表的后缀风格，最后一个是指定分库分表策略



接收配置的类

![1567082900354](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收配置类1.png)



三个数据源的配置实体类

![1567083614407](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收多数据源配置类1.png)

![1567083963625](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收多数据源配置类2.png)

![1567084041982](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收多数据源配置类3.png)

![1567084247126](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收多数据源配置类4.png)

![1567083763191](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-接收多数据源配置类5.png)

<AbstractRoutingDataSource 是Spring提供给我们专门动态切换数据源的，需要自己实现determineCurrentLookuper()方法>，其中的0.1.2是 数量%3个数据源的3取模

配置这个Spring是无法自动切换的，还需要配置一个类



接下来需要一个配置策略类，决定哪个Bean生效

![1567084341122](F:\GitDepository\学习文档\Java\baseImages\AOP-分库分表-配置策略.png)

​		读写分离：