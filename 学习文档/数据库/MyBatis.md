## MyBatis

### 1、占位符

#{ } 和 ${ }

#### a、区别

​	#{ }是预编译处理，MyBatis在处理#{ }时，它会将sql中的#{ }替换为？，然后调用PreparedStatement的

set方法来赋值，传入字符串后，会在值两边加上单引号，如上面的值 “**4,44,514**”就会变成“ **'4,44,514'** ”；

​	${ }是字符串替换， MyBatis在处理${ }时,它会将sql中的${ }替换为变量的值，传入的数据不会加两边加上单

引号。

​	注意：使用${ }会导致sql注入，不利于系统的安全性！

##### 附1：Sql注入

​	SQL注入：就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务

器执行恶意的SQL命令。常见的有匿名登录（在登录框输入恶意的字符串）、借助异常获取数据库信息等。

#### b、应用场合

​	1、#{ }：主要用户获**取DAO中的参数数据**,在映射文件的SQL语句中出现#{}表达式,底层会创建预编译的SQL；

​	2、${ }:主要用于获取配置文件数据**,DAO接口中的参数信息**,当$出现在映射文件的SQL语句中时创建的不是预编译的SQL,而是字符串的拼接,有可能会导致SQL注入问题.所以一般使用$接收dao参数时,这些参数一般是字段名,表名等,例如order by {column}。

​	注：

​		${}获取DAO参数数据时,参数必须使用@param注解进行修饰或者使用下标或者参数#{param1}形式；

​		#{}获取DAO参数数据时,假如参数个数多于一个可有选择的使用@param。

#### c、如果必须要使用不带 ' 的参数

​	使用${ } 传进来的参数确实不带 " ' " ，但是会引发SQL注入的问题，有时候运行环境（比如docker）会自动拦截注入，此时 ${ } 是不生效的。

​	**解决方法：**

​	使用foreach标签

​		foreach标签主要用于构建in条件，他可以在sql中对集合进行迭代。

~~~xml
 <delete id="deleteByPrimaryKey" parameterType="List" >
    delete from t_slup_customer
    where customerId in
    <foreach item="customerId" collection="customerIds" separator="," open="(" close=")" index="">
      #{customerId, jdbcType=VARCHAR}
    </foreach>
  </delete>
~~~

