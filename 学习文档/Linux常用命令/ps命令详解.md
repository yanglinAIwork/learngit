## Linux下ps命令详解

要对系统中进程进行监测控制，查看状态，内存，CPU的使用情况，使用命令：
/bin/ps

（1）ps ：是显示瞬间进程的状态，并不动态连续；

（2）top：如果想对进程运行时间监控，应该用 top 命令；

（3）kill 用于杀死进程或者给进程发送信号；

​	网页版[PS命令详解](https://www.cnblogs.com/softidea/p/5274988.html)

---

#### **常用命令：**

​	**输出窗口参数解释详见《附录2 参数解释》**

1、ps -A <比ps a好用很多啊，虽然都是列出所有的进程> 列出所有的进程

2、x 显示无控制终端的进程；<这个命令在jvm调优时经常用到>

~~~
-bash-4.1$ ps x
  PID TTY      STAT   TIME COMMAND
 7596 ?        Ssl  934:26 ./redis-server 10.7.130.81:6005 [cluster]
 8041 ?        Sl    13:53 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/java -XX:MetaspaceSize=128m -XX:MaxMeta
 8042 ?        S      0:01 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
 9884 ?        Sl   219:39 /slupapp/var/jdk1.8.0_102/bin/java -Xmx512M -Xms512M -server -XX:+UseG1GC -XX:MaxGCPauseMilli
11161 ?        Ssl  1105:27 ./redis-server 10.7.130.81:6006 [cluster]
13894 pts/0    R+     0:00 ps x
16231 ?        Sl   650:45 /slupapp/var/jdk1.8.0_102/bin/java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20
22197 ?        Sl    35:31 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/java -cp bin/h2-1.4.197.jar org.h2.tool
29543 ?        S      0:00 sshd: slupuser@pts/0
29544 pts/0    Ss     0:00 -bash
30055 ?        Sl     6:41 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/java -XX:MetaspaceSize=128m -XX:MaxMeta
30056 ?        S      0:00 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
30147 ?        Sl     9:05 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/java -XX:MetaspaceSize=128m -XX:MaxMeta
30148 ?        S      0:00 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
30395 ?        Sl    80:44 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64//bin/java -Xms256M -Xmx512M -jar /slupapp/v
-
~~~

3、ww 避免详细参数被截断；<经常使用ps时发现command列展示的命令显示不全，加上ww后就能全部显示了，比w好用，w也显示不全>

4、-aux 显示所有包含其他使用者的进程

~~~
-bash-4.1$ ps -aux
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  19360   516 ?        Ss    2018   0:04 /sbin/init
root       613  0.0  0.0      0     0 ?        S     2018  38:02 [vballoon]
root      1307  0.0  0.0  66220   304 ?        Ss    2018   0:08 /usr/sbin/sshd
root      1325  0.0  0.0      0     0 ?        S     2018   8:33 [flush-252:0]
ntp       1393  0.0  0.0  30740   696 ?        Ss    2018  31:51 ntpd -x -u ntp:ntp -p /var/run/ntpd.pid -g
root      1479  0.0  0.0   4060     8 tty6     Ss+   2018   0:00 /sbin/mingetty /dev/tty6
root      1602  0.0  0.0      0     0 ?        S     2018  37:10 [flush-253:1]
slupuser  7596  0.1  0.0 151964  2216 ?        Ssl   2018 934:27 ./redis-server 10.7.130.81:6005 [cluster]
root      8030  0.0  0.0      0     0 ?        S     2018   0:43 [kdmflush]
slupuser  8041  1.1 11.0 3733208 433036 ?      Sl   Jun18  13:56 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/j
slupuser  8042  0.0  0.0  31828   976 ?        S    Jun18   0:01 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
slupuser  9884  0.2  8.8 3066144 348512 ?      Sl   Apr25 219:40 /slupapp/var/jdk1.8.0_102/bin/java -Xmx512M -Xms512M -s
slupuser 11161  0.2  1.5 231836 59156 ?        Ssl   2018 1105:27 ./redis-server 10.7.130.81:6006 [cluster]
root     15805  0.0  0.0      0     0 ?        Z    10:50   0:00 [ls] <defunct>
slupuser 15806  0.0  0.0 110248  1116 pts/0    R+   10:50   0:00 ps -aux
slupuser 16231  5.1 16.3 4432832 643360 ?      Sl   Jun10 650:50 /slupapp/var/jdk1.8.0_102/bin/java -Xmx1G -Xms1G -serve
consul   17143  1.8  0.5  46836 21652 ?        Sl   May29 555:53 consul agent -node=test_10.7.130.81 -bind=10.7.130.81 -
slupuser 30056  0.0  0.0  31828   968 ?        S    Jun18   0:00 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
slupuser 30147  0.8 12.3 3715060 484560 ?      Sl   Jun18   9:08 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64/bin/j
slupuser 30148  0.0  0.0  31828   932 ?        S    Jun18   0:00 /usr/sbin/rotatelogs %Y-%m-%d_%H 28800 480
slupuser 30395  0.1  7.6 3078408 300400 ?      Sl   May09  80:45 /slupapp/var/zulu8.36.0.1-ca-jdk8.0.202-linux_x64//bin/
-
~~~

5、-e 显示所有进程,环境变量<经常搭配组合 ps -ef>

6、ps -参数PID1,PID2 中间无空格、pid用逗号分隔。 查看指定的进程。

#### **以下命令不常用，属于即使用了你也不知道输出来的窗口是啥的那种**

~~~
j 用任务格式来显示进程；
u 按用户名和启动时间的顺序来显示进程；
f 用树形格式来显示进程；
a 显示所有用户的所有进程（包括其它用户）；
r 显示运行中的进程；
-w 显示加宽可以显示较多的资讯（推荐用ww，w显示不全）
-au 显示较详细的资讯（没有-aux好用）
-f 全格式
-h 不显示标题
-l 长格式
-w 宽输出
--sort X[+|-] key [,[+|-] key [,…]] 从SORT KEYS段中选一个多字母键.“+”字符是可选的,因为默认地方向就是按数字升序或者词典顺序，“-”字符是逆序排序（即降序）.
比如： ps -jax -sort=uid,-ppid,+pid. <排序键详见附录1：排序键>
--help 显示帮助信息.
--version 显示该命令地版本信息.
~~~



### 附录1 排序键

~~~
============排序键列表==========================
c cmd   可执行地简单名称 
C cmdline   完整命令行 
f flags   长模式标志 
g pgrp   进程地组ID 
G tpgid   控制tty进程组ID 
j cutime   累计用户时间 
J cstime   累计系统时间 
k utime   用户时间 
K stime   系统时间 
m min_flt   次要页错误地数量 
M maj_flt   重点页错误地数量 
n cmin_flt 累计次要页错误 
N cmaj_flt 累计重点页错误 
o session   对话ID 
p pid   进程ID 
P ppid   父进程ID 
r rss   驻留大小 
R resident 驻留页 
s size   内存大小(千字节) 
S share   共享页地数量 
t tty   tty次要设备号 
T start_time 进程启动地时间 
U uid   UID
u user   用户名
v vsize   总地虚拟内存数量(字节) 
y priority 内核调度优先级
~~~

### 附录2 参数解释

1、au(x) 输出格式 : 
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND

~~~
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
USER: 进程所有者
PID: 进程ID
%CPU: 占用的 CPU 使用率
%MEM: 占用的内存使用率
VSZ: 占用的虚拟内存大小
RSS: 占用的内存大小
TTY: 终端的次要装置号码 (minor device number of tty)
STAT: 进程状态: <详见 附录3 进程状态>
START: 启动进程的时间； 
TIME: 进程消耗CPU的时间；
COMMAND:命令的名称和参数；
~~~

2、ps 输出格式：

PID TTY TIME COMMAND

~~~
PID(进程ID)
TTY(终端名称)
TIME(进程执行时间)
COMMAND(该进程地命令行输入)
---
%CPU、%MEM两个选项
	前者指该进程占用地CPU时间和总时间地百分比;后者指该进程占用地内存和总内存地百分比.
~~~



### 附录3 进程状态

~~~
D 无法中断的休眠状态（通常 IO 的进程）； 
R 正在运行，在可中断队列中； 
S 处于休眠状态，静止状态； 
T 停止或被追踪，暂停执行； 
W 进入内存交换（从内核2.6开始无效）； 
X 死掉的进程； 
Z 僵尸进程不存在但暂时无法消除；
W: 没有足够的记忆体分页可分配
WCHAN 正在等待的进程资源；
<: 高优先级进程
N: 低优先序进程
L: 有记忆体分页分配并锁在记忆体内 (即时系统或捱A I/O)，即,有些页被锁进内存
s 进程的领导者（在它之下有子进程）； 
l 多进程的（使用 CLONE_THREAD, 类似 NPTL pthreads）； 
+ 位于后台的进程组；
~~~

### 附录4 Kill命令信号

~~~
kill 终止进程
有十几种控制进程的方法，下面是一些常用的方法:

kill -STOP [pid] 
发送SIGSTOP (17,19,23)停止一个进程，而并不消灭这个进程。

kill -CONT [pid] 
发送SIGCONT (19,18,25)重新开始一个停止的进程。

kill -KILL [pid] 
发送SIGKILL (9)强迫进程立即停止，并且不实施清理操作。

kill -9 -1 
终止你拥有的全部进程。

SIGKILL 和 SIGSTOP 信号不能被捕捉、封锁或者忽略，但是，其它的信号可以。所以这是你的终极武器。
~~~

