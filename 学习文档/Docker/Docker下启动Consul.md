## Docker下启动Consul

### 一、拉取镜像

```
docker pull consul
```

### 二、启动集群

这里启动4个Consul Agent，3个Server（会选举出一个leader），1个Client。

```
#启动第1个Server节点，集群要求要有3个Server，将容器8500端口映射到主机8900端口，同时开启管理界面
docker run -d --name=consul1  -p 8500:8500 -e CONSUL_BIND_INTERFACE=eth0 --add-host localhost:10.63.203.39 consul agent --server=true --bootstrap-expect=3 --client=0.0.0.0 -ui
 
#启动第2个Server节点，并加入集群
docker run -d --name=consul2 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=true --client=0.0.0.0 --join 172.17.0.5
 
#启动第3个Server节点，并加入集群
docker run -d --name=consul3 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=true --client=0.0.0.0 --join 172.17.0.5
 
#启动第4个Client节点，并加入集群
docker run -d --name=consul4 -e CONSUL_BIND_INTERFACE=eth0 consul agent --server=false --client=0.0.0.0 --join 172.17.0.5
```

第1个启动容器的IP一般是172.17.0.2，后边启动的几个容器IP会排着来：172.17.0.3、172.17.0.4、172.17.0.5。



172.22.224.1    localhost
172.22.224.1    DESKTOP-G544I0V