一、符号部分

1. &lt;  小于号 （**\&lt;**）  &gt;  大于号（**\&gt;**）  

2. ~~~jsp
   &#8729; 在页面显示是第三条
   &copy; 在页面显示是第四条
   ~~~

3.  **&#8729;**  

4. &copy;  

5.  

---

二、代码部分

1. **@SuppressWarnings("all")**   取消全部警告

2. **RequestParam** 只支持 **GET**请求， **RequestBody** 只支持**Post**请求 

3. 生成uuid 

   ~~~java
   UUID.randomUUID().toString()
   ~~~

4. HashMap的key可以为空

   ~~~java
   Map<String, String> map = new HashMap<>();
   map.put(null,"a");
   System.out.println(map.get(null));
   
   //a
   ~~~

5. Java转码

   ~~~java
   String abcUtf8 = "哈哈哈哈";
   String abcUnicode = "";
   try {
   	abcUnicode = new String(abcUtf8.getBytes("UTF-8"),"Unicode");
   } catch (UnsupportedEncodingException e) {
   	e.printStackTrace();
   }
   ~~~

6. 实体类copy<小-大>

```java
BeanUtils.copyProperties(beInsuredInfo,beInsuredBo);
```



/** 

 * 使用jackon的ObjectMapper的writeValueAsString方法可以把java对象转化成json字符串 
 * 输出的结果是：{"array":["80","90","95"],"height":170,"innerBaseObject":{"array":["65","68","75"], 
 * "height":171,"innerBaseObject":null,"userCode":"2号","userName":"李四","weight":75.5,"sex":true}, 
 * "userCode":"1号","userName":"张三","weight":65.5,"sex":true} 
 * 
     */  

SELECT DISTINCT module.moduleCode moduleCode，
case
	         when rprivilege.stanby2 != '' then
	          rprivilege.stanby2
	         else
	          '0'
	       end as authflag
FROM t_slup_customized       customized  WHERE customized.userCode = #{userCode}