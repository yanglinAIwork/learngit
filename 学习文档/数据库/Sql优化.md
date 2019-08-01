## MySql优化

### 1、EXPLAIN

​		explain 查看SQL执行计划

示例：`explain select * from t_slup_tmp`

![1561686813595](D:\Typora文档\文档\学习文档\数据库\Explain.png)

+ **id**，id相同时，执行顺序从上往下执行，id不同时，值越大越先被执行。
+ **select_type**，可以查看id的执行实例，共有以下几种类型：
  + SIMPLE： 表示此查询不包含 UNION 查询或子查询
  + PRIMARY： 表示此查询是最外层的查询
  + SUBQUERY： 子查询中的第一个 SELECT
  + UNION： 表示此查询是 UNION 的第二或随后的查询
  + DEPENDENT UNION： UNION 中的第二个或后面的查询语句, 取决于外面的查询
  + UNION RESULT, UNION 的结果
  + DEPENDENT SUBQUERY: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.
  + DERIVED：衍生，表示导出表的SELECT（FROM子句的子查询）
+ **table**，表示查询涉及的表或衍生的表。
+ **type列**，连接类型，通过type字段，可以判断此次查询是全部扫描还是索引扫描。一个好的SQL语句至少要达到range级别，杜绝出现all级别， type 类型的性能关系如下：`ALL < index < range ~ indexmerge < ref < eqref < const < system`，常见的值有：
  + `system`：表中只有一条数据， 这个类型是特殊的 const 类型。 
  + `const`: 针对主键或唯一索引的等值查询扫描，最多只返回一行数据。 const 查询速度非常快， 因为它仅仅读取一次即可。例如下面的这个查询，它使用了主键索引，因此 type 就是 const 类型的：explain select * from user_info where id = 2；
  + `eqref`： 此类型通常出现在多表的 join 查询，表示对于前表的每一个结果，都只能匹配到后表的一行结果。并且查询的比较操作通常是 =，查询效率较高。例如：`explain select * from userinfo, orderinfo where userinfo.id = orderinfo.userid;`
  + `ref`：此类型通常出现在多表的 join 查询，针对于非唯一或非主键索引，或者是使用了 最左前缀 规则索引的查询。例如下面这个例子中， 就使用到了 ref 类型的查询：`explain select * from userinfo, orderinfo where userinfo.id = orderinfo.userid AND orderinfo.user_id = 5`
  + `rang`：表示使用索引范围查询，通过索引字段范围获取表中部分数据记录。这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中。例如下面的例子就是一个范围查询：`explain select * from user_info  where id between 2 and 8；`
  + `index`：表示全索引扫描(full index scan)，和 ALL 类型类似，只不过 ALL 类型是全表扫描，而 index 类型则仅仅扫描所有的索引， 而不扫描数据。index 类型通常出现在：所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据。当是这种情况时，Extra 字段 会显示 Using index。
  + `all`：表示全表扫描，这个类型的查询是性能最差的查询之一。通常来说， 我们的查询不应该出现 ALL 类型的查询，因为这样的查询在数据量大的情况下，对数据库的性能是巨大的灾难。 如一个查询是 ALL 类型查询， 那么一般来说可以对相应的字段添加索引来避免。
+ **possible_keys**，它表示 mysql 在查询时，可能使用到的索引。 注意，即使有些索引在 possible_keys 中出现，但是并不表示此索引会真正地被 mysql 使用到。 mysql 在查询时具体使用了哪些索引，由 key 字段决定。
+ **key列**，使用到的索引名。如果没有选择索引，值是NULL。可以采取强制索引方式。
+ **key_len列**，表示查询优化器使用了索引的字节数，这个字段可以评估组合索引是否完全被使用。
+ **ref**，这个表示显示索引的哪一列被使用了，如果可能的话,是一个常量。前文的type属性里也有ref，注意区别。
+ **rows列**，扫描行数，该值是个预估值，这个值非常直观的显示 sql 效率好坏， 原则上 rows 越少越好。可以对比key中的例子，一个没建立索引钱，rows是9，建立索引后，rows是4。
+ **extra列**，详细说明。注意，常见的不太友好的值，如下：Using filesort，Using temporary。
  + `Using filesort`：表示 mysql 需额外的排序操作，不能通过索引顺序达到排序效果。一般有 using filesort都建议优化去掉，因为这样的查询 cpu 资源消耗大。
  + `using index`：覆盖索引扫描，表示查询在索引树中就可查找所需数据，不用扫描表数据文件，往往说明性能不错。
  + `Using temporary`：查询有使用临时表, 一般出现于排序， 分组和多表 join 的情况， 查询效率不高，建议优化。
  + `using where`：表名使用了where过滤。

#### 附EXPLAIN优化案例

~~~sql
EXPLAIN SELECT u.*, o.* FROM user_info u LEFT JOIN order_info o ON u.id = o.user_id;
~~~

​		执行结果，type有ALL，并且没有索引：

![img](D:\Typora文档\文档\学习文档\数据库\Explain优化1.png)

​		开始优化，在关联列上创建索引，明显看到type列的ALL变成ref，并且用到了索引，rows也从扫描9行变成了1行：

![img](D:\Typora文档\文档\学习文档\数据库\Explain优化2.png)

​		这里面一般有个规律是：左连接索引加在右表上面，右连接索引加在左表上面。

![img](D:\Typora文档\文档\学习文档\数据库\需要索引的情况.png)

---

### 2、SQL语句中IN包含的值不应过多

​		MySQL对于IN做了相应的优化，即将IN中的常量全部存储在一个数组里面，而且这个数组是排好序的。但是如果数值较多，产生的消耗也是比较大的。再例如：select id from t where num in(1,2,3) 对于连续的数值，能用between就不要用in了；再或者使用连接来替换。

---

### 3、SELECT语句务必指明字段名称

​		SELECT*增加很多不必要的消耗（CPU、IO、内存、网络带宽）；增加了使用覆盖索引的可能性；当表结构发生改变时，前断也需要更新。所以要求直接在select后面接上字段名。

---

### 4、当只需要一条数据的时候，使用limit 1

​		这是为了使EXPLAIN中type列达到const类型

---

### 5、如果排序字段没有用到索引，就尽量少排序

### 6、如果限制条件中其他字段没有索引，尽量少用or

​		or两边的字段中，如果有一个不是索引字段，而其他条件也不是索引字段，会造成该查询不走索引的情况。很多时候使用union all或者是union（必要的时候）的方式来代替“or”会得到更好的效果。

### 7、尽量用union all代替union

​		union和union all的差异主要是前者需要将结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的CPU运算，加大资源消耗及延迟。当然，union all的前提条件是两个结果集没有重复数据。

### 8、不使用ORDER BY RAND()

~~~sql
select id from `dynamic` order by rand() limit 1000;
~~~

优化后

~~~sql
select id from `dynamic` t1 join (select rand() * (select max(id) from `dynamic`) as nid) t2 on t1.id > t2.nidlimit 1000;
~~~

### 9、区分in和exists、not in和not exists

~~~sql
select * from 表A where id in (select id from 表B)

--上面SQL语句相当于
select * from 表A where exists(select * from 表B where 表B.id=表A.id)
~~~

​		区分in和exists主要是造成了驱动顺序的改变（这是性能变化的关键），如果是exists，那么以外层表为驱动表，先被访问，如果是IN，那么先执行子查询。所以IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。

​		关于not in和not exists，推荐使用not exists，不仅仅是效率问题，not in可能存在逻辑问题。如何高效的写出一个替代not exists的SQL语句？

~~~sql
--原SQL语句：
select colname … from A表 where a.id not in (select b.id from B表)

--高效的SQL语句：
select colname … from A表 Left join B表 on where a.id = b.id and b.id is null
~~~

取出的结果集如下图表示，A表不在B表中的数据：

![1561717021164](D:\Typora文档\文档\学习文档\数据库\Left join结果集.png)

### 10、使用合理的分页方式以提高分页的效率

~~~sql
select id,name from product limit 866613, 20

/*
使用上述SQL语句做分页的时候，可能有人会发现，随着表数据量的增加，直接使用limit分页查询会越来越慢。
优化的方法如下：可以取前一页的最大行数的id，然后根据这个最大的id来限制下一页的起点。比如此列中，上一页最大的id是866612。SQL可以采用如下的写法
*/

select id,name from product where id> 866612 limit 20
~~~

### 11、分段查询

​		在一些用户选择页面中，可能一些用户选择的时间范围过大，造成查询缓慢。主要的原因是扫描行数过多。这个时候可以通过程序，分段进行查询，循环遍历，将结果合并处理进行展示。

![1561717180484](D:\Typora文档\文档\学习文档\数据库\分段查询.png)

### 12、避免在where子句中对字段进行null值判断

​		对于null的判断会导致引擎放弃使用索引而进行全表扫描。

### 13、不建议使用%前缀模糊查询

​		例如LIKE“%name”或者LIKE“%name%”，这种查询会导致索引失效而进行全表扫描。但是可以使用LIKE “name%”。

​	那如何查询%name%？如下图所示，虽然给secret字段添加了索引，但在explain结果并没有使用：

![1561717261559](D:\Typora文档\文档\学习文档\数据库\前缀模糊查询1.png)

​		那么如何解决这个问题呢，答案：使用全文索引。

​		在我们查询中经常会用到`select id,fnum,fdst from dynamic_201606 where user_name like '%zhangsan%'; `这样的语句，普通索引是无法满足查询需求的。庆幸的是在MySQL中，有全文索引来帮助我们。

​		创建全文索引的SQL语法是：

~~~sql
ALTER TABLE `dynamic_201606` ADD FULLTEXT INDEX `idx_user_name` (`user_name`);

--使用全文索引的SQL语句是：
select id,fnum,fdst from dynamic_201606 where match(user_name) against('zhangsan' in boolean mode);
~~~

​		**注意：**在需要创建全文索引之前，请联系DBA确定能否创建。同时需要注意的是查询语句的写法与普通索引的区别。

### 14、避免在where子句中对字段进行表达式操作

~~~sql
select user_id,user_project from user_base where age*2=36;

--对字段就行了算术运算，这会造成引擎放弃使用索引，建议改成：
--优化后
select user_id,user_project from user_base where age=36/2;
~~~

### 15、避免隐式类型转换

​		where子句中出现column字段的类型和传入的参数类型不一致的时候发生的类型转换，建议先确定where中的参数类型。

### 16、对于联合索引来说，要遵守最左前缀法则

​		举列来说索引含有字段id、name、school，可以直接用id字段，也可以id、name这样的顺序，但是name;school都无法使用这个索引。所以在创建联合索引的时候一定要注意索引字段顺序，常用的查询字段放在最前面。

### 17、必要时可以使用force index来强制查询走某个索引

​		有的时候MySQL优化器采取它认为合适的索引来检索SQL语句，但是可能它所采用的索引并不是我们想要的。这时就可以采用forceindex来强制优化器使用我们制定的索引。

### 18、注意范围查询语句

​		对于联合索引来说，如果存在范围查询，比如between、>、<等条件时，会造成后面的索引字段失效。

### 19、关于JOIN优化<有专门一节介绍>

![1561717493377](D:\Typora文档\文档\学习文档\数据库\Join示例图.png)

​		LEFT JOIN A表为驱动表，INNER JOIN MySQL会自动找出那个数据少的表作用驱动表，RIGHT JOIN B表为驱动表。

​		**注意：**

**1）MySQL中没有full join，可以用以下方式来解决：**

~~~sql
select * from A left join B on B.name = A.namewhere B.name is nullunion allselect * from B;
~~~

**2）尽量使用inner join，避免left join：**

​		参与联合查询的表至少为2张表，一般都存在大小之分。如果连接方式是inner join，在没有其他过滤条件的情况下MySQL会自动选择小表作为驱动表，但是left join在驱动表的选择上遵循的是左边驱动右边的原则，即left join左边的表名为驱动表。

**3）合理利用索引：**

​		被驱动表的索引字段作为on的限制字段。

**4）利用小表去驱动大表：**

​		![1561717586550](D:\Typora文档\文档\学习文档\数据库\小表驱动大表.png)

​		从原理图能够直观的看出如果能够减少驱动表的话，减少嵌套循环中的循环次数，以减少 IO总量及CPU运算的次数。

**5）巧用STRAIGHT_JOIN：**

​		inner join是由MySQL选择驱动表，但是有些特殊情况需要选择另个表作为驱动表，比如有group by、order by等「Using filesort」、「Using temporary」时。STRAIGHT_JOIN来强制连接顺序，在STRAIGHT_JOIN左边的表名就是驱动表，右边则是被驱动表。在使用STRAIGHT_JOIN有个前提条件是该查询是内连接，也就是inner join。其他链接不推荐使用STRAIGHT_JOIN，否则可能造成查询结果不准确。

![1561717630578](D:\Typora文档\文档\学习文档\数据库\STRAIGHTJOIN.png)

这个方式有时能减少3倍的时间。