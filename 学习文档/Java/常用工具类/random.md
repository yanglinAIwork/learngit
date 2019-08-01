## random随机数

1、生成100-999的随机数

~~~java
// 100-999的随机数  
int random = (int) (Math.random() * 1000);  
while (random < 100) {  
    random = random * 10;  
}  
~~~

以上的代码为有bug的，如果Math.random值小于0.001，int 强转之后值为0。while会陷入死循环

修改后的代码

~~~java
// 100-999的随机数  
int random = new java.util.Random().nextInt(900) + 100;
~~~



```java
    UUID.randomUUID().toString().replace("-", "")
```