## Docker Network

​	docker 提供给我们多种(4种)网络模式，我们可以根据自己的需求来使用。例如我们在一台主机（host）或者同一个docker engine上面运行continer的时候，我们就可以选择bridge网络模式；而当我们需要在多台host上来运行多个container来协同工作的时候，overlay模式就是我们的首选。

​	当我们完成docker engine的安装以后，docker会在每一个engine上面生成一个3种网络，他们是：bridge, none 还有host。

### 一、默认网络模式 - bridge

​	首先来侃一侃docker0. 之所以说它是默认的网络，是由于当我们运行container的时候没有“显示”的指定网络时，我们的运行起来的container都会加入到这个“默认” docker0 网络。他的模式是bridge。

### 二、无网络模式 - none

​	顾名思义，所有加入到这个网络模式中的container，都"不能”进行网络通信。貌似有点鸡肋

### 三、宿主网络模式 - host

​	这种网络模式将container与宿主机的网络相连通，虽然很直接，但是却破获了container的隔离性，因此也比较鸡肋。

### 四、自定义网络

​	由于之前介绍的3种自带的网络模式有各自的局限性，因此，docker推荐大家自定义网络。通过自定义网络，我们可以实现“服务发现”与“DNS解析”。

​	docker 允许我们创建3种类型的自定义网络，bridge，overlay，MACVLAN。

#### 1、自定义bridge网络

​	与docker0类似，我们可以自定义bridge网络，通过使用自定义bridge网络，我们就可以实现在一台host上的多个container之间的通信。他的网络模型如下(图片来自docker官网):

![img](D:\Typora文档\文档\学习文档\Docker\DockerNetworkBridge.png)

#### 2、docker_gwbridge

​	他在本质上还是一个local的bridge网络，但是他是我们实现多个host之间的container通信的基础。通常情况下，当我们在链接swarm nodes的时候，docker_gwbridge网络就会被在每一个swarm节点上自动创建出来。

#### 3、自定义Overlay网络

​	overlay网络用于连接不同机器上的docker容器，允许不同机器上的容器相互通信，同时支持对消息进行加密，当我们初始化一个swarm或是加入到一个swarm中时，在docker主机上会出现两种网络：

​	swarm在设计之初是为了service(一组container)而服务的，因此通过swarm创建的overlay网络在一开始并不支持单独的container加入其中。但是在docker1.13, 我们可以通过“--attach” 参数声明当前创建的overlay网络可以被container直接加入。

​	（1）称为ingress的overlay网络，用于传递集群服务的控制或是数据消息，若在创建swarm服务时没有指定连接用户自定义的overlay网络，将会加入到默认的ingress网络

​	（2）名为docker_gwbridge桥接网络会连接swarm中所有独立的docker系统进程

- 注意事项：
- 如果想要连接到overlay网络，请确保连接前下列端口没有服务，并且服务器防火墙要允许下列端口通过：

1. TCP端口2377，用于集群管理信息的交流
2. TCP、UDP端口7946用于集群中节点的交流
3. UDP端口4789用于overlay网络中数据报的发送与接收

~~~
docker network create --driver=overlay --attachable name=myOverlayNet 
~~~



### 这一章本地单个机器无法操作测试，请自行百度。案例很多！！！

