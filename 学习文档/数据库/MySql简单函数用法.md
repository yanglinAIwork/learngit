## MySql简单函数用法

### 1.Union all/Union

​		在开发中，有些数据的存储可能涉及到分库分表，查询的时候，可能需要查询所有的分表，这个时候，就需要用到UNION或者UNION ALL。用于合并两个或多个SELECT语句的结果集

​		注意：Union 内部的语句必须有相同数量的列，列也必须有相似的数据类型，同时每个select语句中列的顺序也必须相同。Union 不会出现重复的结果集，Union all 会出现重复的结果集

语法：

~~~SQL
select t.customerId from t_slup_customer t 
UNION SELECT n.customerId from t_slup_agent_customer_relation n ;
~~~

---

### 2.Join 

​		**条件为 on**

#### a、Inner Join<内连接>

​		内连接INNER JOIN是最常用的连接操作，求两个表的交集。

​		有四种写法：

~~~sql
SELECT * FROM t_blog INNER JOIN t_type ON t_blog.typeId=t_type.id;
SELECT * FROM t_blog,t_type WHERE t_blog.typeId=t_type.id;
SELECT * FROM t_blog STRAIGHT_JOIN t_type ON t_blog.typeId=t_type.id; --注意STRIGHT_JOIN有个下划线
SELECT * FROM t_blog JOIN t_type ON t_blog.typeId=t_type.id;
~~~

#### b、CROSS JOIN<笛卡尔积>

​		笛卡尔积就是将A表的每一条记录与B表的每一条记录强行拼在一起。所以，如果A表有n条记录，B表有m条记录，笛卡尔积产生的结果就会产生n*m条记录。下面的例子，t_blog有10条记录，t_type有5条记录，所有他们俩的笛卡尔积有50条记录。

​		有五种产生笛卡尔积的方式如下

~~~sql
SELECT * FROM t_blog CROSS JOIN t_type;
SELECT * FROM t_blog INNER JOIN t_type;
SELECT * FROM t_blog,t_type;
SELECT * FROM t_blog NATURE JOIN t_type;
select * from t_blog NATURA join t_type;
~~~

#### c、LEFT JOIN<左连接>

​		左连接LEFT JOIN的含义就是求两个表的交集外加左表剩下的数据。

~~~SQL
SELECT * FROM t_blog LEFT JOIN t_type ON t_blog.typeId=t_type.id;
~~~

#### d、RIGHT JOIN<右连接>

​		同 2.c 相反

#### e、OUTER JOIN<外连接>

​		外连接就是求两个集合的并集，即取两个表不同的数据。MySQL不支持OUTER JOIN

~~~sql
SELECT * FROM t_blog LEFT JOIN t_type ON t_blog.typeId=t_type.id
UNION
SELECT * FROM t_blog RIGHT JOIN t_type ON t_blog.typeId=t_type.id;
~~~

f

g、

#### USING子句

​		MySQL中连接SQL语句中，ON子句的语法格式为：table1.column_name = table2.column_name。当模式设计对联接表的列采用了相同的命名样式时，就可以使用 USING 语法来简化 ON 语法，格式为：USING(column_name)。

​		区别：USING指定一个属性名用于连接两个表，而ON指定一个条件。另外，SELECT *时，USING会去除USING指定的列，而ON不会<即a表有 n，m，l 三列，b表有n，o，p 三列，on作为条件时结果集为n，m，l，n，o，p 六列，而USING作为条件时结果集只有 n，m，l，o，p 五列>。

~~~sql
 SELECT * FROM t_blog INNER JOIN t_type ON t_blog.id =t_type.id;
 SELECT * FROM t_blog INNER JOIN t_type USING(id);
 --两种写法是等价的，但列数目不同
~~~

****

### 3.distinct



### 4.decode



