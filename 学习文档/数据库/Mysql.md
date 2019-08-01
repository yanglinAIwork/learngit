### Windows 版本：

#### 1、配置环境

​	在环境变量配置 **MYSQL_HOME** 及其目录（eg: C:\Program Files\MySQL\MySQL Server 5.7）

​	在path中配置 %MYSQL_HOME%\bin

#### 2、本地启动Mysql命令：

​	mysql -u root -p

#### 3、where ‘1’ = ‘1’的使用

​	Mybatis的xml文件中where条件尽量加上 '1'='1'，否则当where <if></if>中的值都为空时，只存在一个where，后面没有条件，sql语句明显会有问题。

#### 4、null 字段

字段尽量少用null，如下示例：

数据库表字段：

| id   | appntidtype | insured_name |
| ---- | ----------- | ------------ |
| 1    | 0           | 贾斯丁       |
| 2    | 2           | 贾斯丁       |
| 3    | null        | 贾斯丁       |

##### 错误1

​	**查询语句：**

~~~sql
SELECT * from t_slup_wechatuser where appntidtype not in ('1','2');
~~~

查询结果：

| id   | appntidtype | insured_name |
| ---- | ----------- | ------------ |
| 1    | 0           | 贾斯丁       |

很明显的错误，没有查到 null 的那一行

##### 错误2

​	计数

~~~
SELECT count(appntidtype) from t_slup_wechatuser;
~~~

结果为2，null不计入统计（很明显想要的结果是3）

##### 错误3

​	条件判空

~~~sql
select * from t_slup_wechatuser where appntidtype = null
~~~

​	这个得不到结果，需要修改为

~~~sql
select * from t_slup_wechatuser where appntidtype is null
~~~

#### 5、UUID字段

```sql
insert into tableName(id, ……) values(REPLACE(UUID(),'-',''),……);
```

#### 6、Msql在指定列名前/后新增列

~~~sql
alter table t_slup_agent add column branchtype2 varchar(9) DEFAULT '' COMMENT '代理人所属子渠道:03:中心城市人' after branchtype;	
~~~

#### 7、事务

https://blog.csdn.net/nextyu/article/details/78669997

#### 8、查询多个字段重复数据

~~~sql
select name,sex,birthday,idtype,idno,count(*) from t_slup_customer group by name,sex,birthday,idtype,idno having count(*) > 1
~~~

#### 9、在两个表中查询重复的数据

​	背景：a表有customerId字段和userCode字段，b表有customerId字段和mobile字段，要根据userCode和mobile字段查两个表是否有重复的customerId字段

​	说明：a表customerId和userCode两个字段可以作为关联主键，但是customerId和userCode是多对多关系，b表同样如此

​	如果直接使用

~~~sql
select m.customerId,m.mobile,n.usercode from t_slup_customer m, t_slup_agent_customer_relation n where m.customerId = n.customerId 
~~~

​	是有问题的，自己测试吧，反正数据对不上

~~~sql
/*给了手机号 +userCode 查两表重复数据*/
/*以下是设计思路 及验证方案，具体实现方法在4*/
/*1.通过userCode 查出所有的 customerId */
/*2.通过手机号查出所有的 customerId*/
select m.customerId from t_slup_customer m, t_slup_agent_customer_relation n where m.mobile = ? and n.usercode = ?

/*1.查询t_slup_agent_customer_relation所有的distinct数据*/
select DISTINCT a.customerId,a.usercode from t_slup_agent_customer_relation a where a.customerId is not null and a.usercode is not null and  a.customerId!= '' and a.usercode!= '' order by a.customerId,a.usercode desc

/*2.查手机号对应的所有的 customerId*/
select m.customerId,m.mobile from t_slup_customer m where m.mobile is not null and m.mobile != '' and LENGTH(m.mobile) > 5 and m.mobile not like '+%' and m.mobile not like '*%';

/*3.查询1对应的所有手机号 */
select DISTINCT a.customerId,a.usercode,b.mobile from t_slup_agent_customer_relation a, t_slup_customer b where a.customerId is not null and a.usercode is not null and  a.customerId!= '' and a.usercode!= '' 
and a.customerId = b.customerId order by a.customerId,a.usercode desc

/*4.分类 分组前 38314 分组后 27246 */ 
select t.customerId,t.usercode,t.mobile,COUNT(1) num from 
(
select DISTINCT a.customerId,a.usercode,b.mobile from t_slup_agent_customer_relation a, t_slup_customer b where a.customerId is not null and a.usercode is not null and  a.customerId!= '' and a.usercode!= '' 
and a.customerId = b.customerId and b.mobile is not null and b.mobile != '' order by a.usercode,a.customerId asc 
) t
GROUP BY t.usercode,t.mobile ORDER BY num desc;


select DISTINCT a.customerId,a.usercode,b.mobile from t_slup_agent_customer_relation a, t_slup_customer b where a.customerId is not null and a.usercode is not null and  a.customerId!= '' and a.usercode!= '' 
and a.customerId = b.customerId and b.mobile is not null and b.mobile != '' order by a.usercode,a.customerId asc ;

/*验证方法 1.建分组前数据表;2.在表里把分组后的数据条数大于一条的userCode和mobile放在临时表查看*/
create table t_slup_tmp as (
select DISTINCT a.customerId,a.usercode,b.mobile from t_slup_agent_customer_relation a, t_slup_customer b where a.customerId is not null and a.usercode is not null and  a.customerId!= '' and a.usercode!= '' 
and a.customerId = b.customerId and b.mobile is not null and b.mobile != '' order by a.usercode,a.customerId asc ;
)

CAST(SUBSTR(m.mobile,1,2) AS INTEGER)< 0


select CAST(SUBSTR(m.mobile,1,2) as SIGNED INTEGER) from t_slup_customer m where m.mobile is not null and m.mobile != '';
select SUBSTR(m.mobile,1,2) from t_slup_customer m where m.mobile is not null and m.mobile != '';
select mobile from t_slup_customer m where m.mobile is not null and m.mobile != '' and m.mobile not like '+%' and m.mobile not like '*%';
~~~

