鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，需要配置加速器来解决，可以使用的是网易的镜像地址：http://hub-mirror.c.163.com。


新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：

{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
具体过程：
vim /etc/docker/daemon.json

若因权限无法修改，加上sudo，即sudo vim /etc/docker/daemon.json，保存并退出后，重新运行docker run hello-world





