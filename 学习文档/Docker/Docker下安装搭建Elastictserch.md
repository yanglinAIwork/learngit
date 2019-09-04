## Docker 安装 ElasticSearch

​	对外暴露端口：9100，9200

### 1、查找镜像

~~~
docker search Elasticsearch	
~~~

### 2、下载镜像

~~~
docker pull elasticsearch
~~~

​	默认下载最新版本的镜像

​	**如果出现以下错误：**

~~~
PS C:\WINDOWS\system32> docker pull elasticsearch
Using default tag: latest
Error response from daemon: manifest for elasticsearch:latest not found
~~~

​	**解决方案：**

​	去 [Docker Hub](https://hub.docker.com/r/library/) 官网查找对应的tag，然后再下载即可

### 3、查看镜像

~~~
PS C:\WINDOWS\system32> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
openzipkin/zipkin   latest              46e18aaff5ac        12 days ago         260MB
elasticsearch       6.7.2               2982ba071059        2 weeks ago         863MB
consul              latest              9c9974471250        2 weeks ago         108MB
~~~

### 4、运行镜像

​	ElasticSearch的默认端口是9200，我们把宿主环境9200端口映射到Docker容器中的9200端口，就可以访问到Docker容器中的ElasticSearch服务了，同时我们把这个容器命名为es。

​	5.0*默认*分配jvm空间大小为*2g* 5.0之前好像是1g

​	9200是restapi访问的端口,9300是java访问的端口

~~~
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" elasticsearch:6.7.2
~~~

### 5、进行配置

#### a、进入容器

~~~
docker exec -it es /bin/bash
~~~

#### b、修改配置

​	配置跨域 (不加页面访问不了9200端口)

~~~
cd config
vi elasticsearch.yml

加入：

http.cors.enabled: true 
http.cors.allow-origin: "*" 
~~~

| 参数                     | 缺省值    | 说明                                                         |
| ------------------------ | --------- | ------------------------------------------------------------ |
| http.cors.enabled        | true      | 如果启用了 HTTP 端口，那么此属性会指定是否允许跨源 REST 请求。 |
| http.cors.allowed.origin | localhost | 如果 http.cors.enabled 的值为 true，那么该属性会指定允许 REST 请求来自何处。 |

#### c、访问测试

~~~json
127.0.0.1:9200

返回
{
    name: "zNSUewe",
    cluster_name: "docker-cluster",
    cluster_uuid: "ngN787fvSEiFIvnpgrC8nQ",
    version: {
        number: "6.7.2",
        build_flavor: "default",
        build_type: "docker",
        build_hash: "56c6e48",
        build_date: "2019-04-29T09:05:50.290371Z",
        build_snapshot: false,
        lucene_version: "7.7.0",
        minimum_wire_compatibility_version: "5.6.0",
        minimum_index_compatibility_version: "5.0.0"
    },
	tagline: "You Know, for Search"
}
~~~

---

## Docker 部署 ElasticSearch-Head

​	为什么要安装ElasticSearch-Head呢，原因是需要有一个管理界面进行查看ElasticSearch相关信息 

​	默认端口 9100

### 1、查找镜像

​	木有认出哪个是要的镜像，可能是 mobz/elasticsearch-head

### 2、下载镜像

~~~
docker pull mobz/elasticsearch-head:5
~~~

​	elasticsearch-head 5可以匹配ES v5、v6

### 3、查看镜像

​	略

### 4、运行镜像

~~~
docker run -d --name es_admin -p 9100:9100 mobz/elasticsearch-head:5 
~~~

### 5、版本出现的错误

​	elasticsearch-head插件在安装x-pack插件之后就无法使用，原因是因为elastic公司在x-pack中加入了安全模块(security机制),就会出现下面的情况，如图：

![1558076848157](F:\GitDepository\学习文档\Docker\images\elasticsearch-head-error.png)

​	解决方案：

​	这个时候需要在elasticseach.yml中增加下面几行配置即可解决。

~~~
http.cors.enabled: true 
http.cors.allow-origin: "*" 
http.cors.allow-headers: "Authorization"
~~~

​	以上第三行可以换为

~~~
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
~~~

​	**然后重启 Elasticsearch 服务即可**

### 6、访问测试

~~~
http://127.0.0.1:9100/
~~~

---

## 安装插件

​	插件官网 https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v6.7.2

### 安装ik和pinyin（安装失败）

#### 1、进入容器：

~~~
docker exec -it es /bin/bash
~~~

#### 2、安装ik，pinyin插件

​	**ik分词器** 、 **pinyin** 搜索插件

~~~
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.2/elasticsearch-analysis-ik-6.3.2.zip

elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v6.7.2/elasticsearch-analysis-ik-6.7.2.zip

elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases

elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.3.2/elasticsearch-analysis-pinyin-6.3.2.zip
~~~

#### 3、exit退出容器、并重启容器：

~~~
docker restart es 
~~~

4、查看插件

~~~
浏览器输入：
http://192.168.99.100:9200/_cat/plugins 
~~~

---

## 安装kibana

### 1、下载镜像

~~~
docker pull kibana:6.7.2
~~~

### 2、查看并运行镜像

~~~
PS C:\WINDOWS\system32> docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
kibana                    6.7.2               2110eecf5f67        2 weeks ago         728MB
elasticsearch             6.7.2               2982ba071059        2 weeks ago         863MB
mobz/elasticsearch-head   5                   b19a5c98e43b        2 years ago         824MB

docker run -it -d -e ELASTICSEARCH_URL=http://172.17.0.2:9200 --name kibana -p 5601:5601 kibana:6.7.2
~~~

### 3、访问测试

~~~
http://127.0.0.1:5601
~~~

## 附录

### 莫名其妙的配置：

```
cluster.name:  //集群名称。如果想让多个节点加入一个集群，那么需要使集群名称一致。
node.name: //节点名，为这个节点起一个独一无二的名字
node.master: //该节点是否担任master角色
node.data: //该节点是否可以担任data节点的角色
network.host: //设置为0.0.0.0 ，意思是任何IP都可以访问
network.publish_host: //本节点在外部的IP
discovery.zen.minimum_master_nodes: //自动发现master节点的最小数，如果这个集群中配置进来的master节点少于这个数目，es的日志会一直报master节点数目不足。
discovery.zen.ping.unicast.hosts: // 按照我的理解，这里配置的host ip才是可以ping通的，因此在这里加上49的ip。因为本文是只有两个节点的es集群，所以只写对方的ip即可。如果是大于2个以上的节点的es集群，那么我想应该是在这里写上所有集群的ip
在这个配置中，注意到这个节点既是主节点又是数据节点，实际上对这个节点的压力是挺大的，在资源比较充裕的条件下不建议这样做。
```

