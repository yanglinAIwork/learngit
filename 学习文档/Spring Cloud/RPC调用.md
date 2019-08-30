一个项目里面有两个以上的业务代码Service，调用的时候属于LPC调用<即本地调用>

一个进程中两个线程的互相调用



两个服务器的不同业务代码调用，属于RPC调用<远程过程调用>

不同进程间的调用



![1566995030938](F:\GitDepository\学习文档\Spring Cloud\RPC调用.png)

调用会涉及网络



RPC是什么

1、希望像调用本地函数一样调用远程接口<在Client端>

~~~java
//不显示IP、端口的调用方式
HelloService h = new HelloService ();

h.hello();
~~~





Dubbo是 RPC框架 + 服务治理框架