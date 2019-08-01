## Docker和docker compose区别

​	docker 的使用过程，它分为镜像构建与容器启动。

​	镜像构建：即创建一个镜像，它包含安装运行所需的环境、程序代码等。这个创建过程就是使用 dockerfile 来完成的。

​	容器启动：容器最终运行起来是通过拉取构建好的镜像，通过一系列运行指令（如端口映射、外部数据挂载、环境变量等）来启动服务的。针对单个容器，这可以通过 docker run 来运行。

​	而如果涉及多个容器的运行（如服务编排）就可以通过 docker-compose 来实现，它可以轻松的将多个容器作为 service 来运行（当然也可仅运行其中的某个），并且提供了 scale (服务扩容) 的功能。

​	**简单总结：**

 	1. dockerfile: 构建镜像；
 	2. docker run: 启动容器；
 	3. docker-compose: 启动服务；
 	4. 但是假如要启动Nacos、Mysql等镜像，Docker的话需要一个个启动，Docker-compose可以一次性全启动。

**举例子：**

​	**问1：**

​	假如你不用 docker ，搭建 wordpress 怎么弄？

​	**答1：**

​	先找台 server ，假设其 OS 为 Ubuntu ，然后按照文档一步步敲命令，写配置，对吧？用 docker 呢？ 随便找台 server ，不管什么操作系统，只要支持 docker 就行，docker run ubuntu, docker 会从官方源里拉取最新的 Ubuntu 镜像，可以认为你开了个 Ubuntu 虚拟机，然后一步步安装，跟上面一样。

 	**缺陷**：

​	但是这样安装有个显著的缺点，一旦 container 被删，你做的工作就都没了。当然可以用 docker commit 来保存成镜像，这样就可以复用了。

​	但是镜像一般比较大，而且只分享镜像的话，别人也不知道你这镜像到底包含什么，这些问题都不利于分享和复用。

​	**缺陷解决方法：**

​	一个直观的解决方案就是，写个脚本把安装过程全部记录下来，这样再次安装的时候，执行脚本就行了。 Dockerfile 就是这样的脚本，它记录了一个镜像的制作过程。

​	有了 Dockerfile, 只要执行 docker build . 就能制作镜像，而且 Dockerfile 就是文本文件，修改也很方便。现在有了 wordpress 的镜像，只需要 docker run 就把 wordpress 启动起来了。

​	**缺陷2：**

​	如果仅仅是 wordpress, 这也就够了。但是很多时候，需要多个镜像合作才能启动一个服务，比如前端要有 

nginx ， 数据库 mysql, 邮件服务等等，当然你可以把所有这些都弄到一个镜像里去，但这样做就无法复用了。

更常见的是, nginx, mysql, smtp 都分别是个镜像，然后这些镜像合作，共同服务一个项目。

​	**缺陷2解决方法：**

​	docker-compose 就是解决这个问题的。你的项目需要哪些镜像，每个镜像怎么配置，要挂载哪些 volume, 等等信息都包含在 docker-compose.yml 里。要启动服务，只需要 docker-compose up 就行，停止也只需要 docker-compse stop/down

​	简而言之， Dockerfile 记录单个镜像的构建过程， docker-compse.yml 记录一个项目(project, 一般是多个镜像)的构建过程。

​	**疑问1：**

​	有些教程用了 dockerfile+docker-compose

​	**疑问1解答：**

​	因为 docker-compose.yml 本身没有镜像构建的信息，如果镜像是从 docker registry 拉取下来的，那么 Dockerfile 就不需要；如果镜像是需要 build 的，那就需要提供 Dockerfile.

​	docker-compose是编排容器的。例如，你有一个php镜像，一个mysql镜像，一个nginx镜像。如果没有docker-compose，那么每次启动的时候，你需要敲各个容器的启动参数，环境变量，容器命名，指定不同容器的链接参数等等一系列的操作，相当繁琐。而用了docker-composer之后，你就可以把这些命令一次性写在docker-composer.yml文件中，以后每次启动这一整个环境（含3个容器）的时候，你只要敲一个docker-composer up命令就ok了。

​	**区别：**

​	dockerfile的作用是从无到有的构建镜像。它包含安装运行所需的环境、程序代码等。这个创建过程就是使用dockerfile 来完成的。Dockerfile为 docker build 命令准备的，用于建立一个独立的 image ，在 docker-compose 里也可以用来实时 builddocker-compose.yml - 为 docker-compose 准备的脚本，可以同时管理多个 container ，包括他们之间的关系、用官方 image 还是自己 build 、各种网络端口定义、储存空间定义等