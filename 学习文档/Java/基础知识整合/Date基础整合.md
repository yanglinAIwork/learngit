## Date类

### 一、生成时间戳

​	long methodStartTime = System.currentTimeMillis();

### 二、时间格式化

~~~java
SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmssSSS");
String time = sdf.format(new java.util.Date());	
~~~

