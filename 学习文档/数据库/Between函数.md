### Between函数

#### 一、对于字符串

Between在使用时，对字符串而言包括前面，不包括后面。示例：

Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |
| 4    | Gates    | Bill      | Xuanwumen 10   | Beijing  |

1、如需以字母顺序显示介于 "Adams"（包括）和 "Carter"（不包括）之间的人，请使用下面的 SQL：

```sql
SELECT * FROM Persons
WHERE LastName
BETWEEN 'Adams' AND 'Carter'
```

结果集：

| Id   | LastName | FirstName | Address       | City     |
| :--- | :------- | :-------- | :------------ | :------- |
| 1    | Adams    | John      | Oxford Street | London   |
| 2    | Bush     | George    | Fifth Avenue  | New York |

**重要事项：**不同的数据库对 BETWEEN...AND 操作符的处理方式是有差异的。某些数据库会列出介于 "Adams" 和 "Carter" 之间的人，但不包括 "Adams" 和 "Carter" ；某些数据库会列出介于 "Adams" 和 "Carter" 之间并包括 "Adams" 和 "Carter" 的人；而另一些数据库会列出介于 "Adams" 和 "Carter" 之间的人，包括 "Adams" ，但不包括 "Carter" 。

所以，请检查你的数据库是如何处理 BETWEEN....AND 操作符的！

---

2、如需使用上面的例子显示范围之外的人，请使用 NOT 操作符：

```sql
SELECT * FROM Persons
WHERE LastName
NOT BETWEEN 'Adams' AND 'Carter'
```

结果集：

| Id   | LastName | FirstName | Address        | City    |
| :--- | :------- | :-------- | :------------- | :------ |
| 3    | Carter   | Thomas    | Changan Street | Beijing |
| 4    | Gates    | Bill      | Xuanwumen 10   | Beijing |

---

#### 二、对于时间格式

Between在对事件进行判断时，查询出的结果既包括前面的时间，又包括后面的时间。

原表

| modifydate |
| ---------- |
| 17:39:00   |
| 17:39:08   |
| 17:39:20   |
| 17:39:25   |

~~~sql
SELECT DATE_FORMAT(modifydate,'%H:%i:%s') from t_slup_interface
where DATE_FORMAT(modifydate,'%H:%i:%s') 
between '17:39:00' and '17:39:20';
~~~

结果集：

| DATE_FORMAT(modifydate,'%H:%i:%s') |
| ---------------------------------- |
| 17:39:08                           |
| 17:39:20                           |

