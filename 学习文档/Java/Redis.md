#### 1、链接远程

​	redis： redis-cli -h 172.28.28.127 -p 6379

​	AUTH password

​	get KEY

---

#### 2、开启redis远程连接

​	1）修改redis服务器的配置文件

~~~
vi redis.conf

注释以下绑定的主机地址

bind 127.0.0.1

或

vim  redis.conf
bind  0.0.0.0
protected-mode   no
~~~

​	2）修改redis服务器的参数配置
​	修改redis的守护进程为no，不启用

~~~
127.0.0.1:6379> config  set   daemonize  "no"
OK

或者

config set requirepass 123 ->123是密码
注意：开启 6379端口
~~~

---

#### 3、