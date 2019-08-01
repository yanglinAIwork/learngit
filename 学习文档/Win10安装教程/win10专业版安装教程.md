#### 1、获取win10专业版镜像

​	本篇所使用的镜像为 **cn_windows_10_consumer_editions_version_1803_updated_march_2018_x64_dvd_12063766.iso** 文件

---

#### 2、右击文件，如同

​	![2019-04-22_105101](D:\Typora文档\文档\学习文档\Win10安装教程\2019-04-22_105101.png)

---

#### 3、在界面输入激活码：

​	J2WWN-Q4338-3GFW9-BWQVK-MG9TT （若不可用可自行百度）

---

#### 4、傻瓜式下一步即可



#### 5、激活

​	1）系统安装完毕后，首先以管理员身份打开CMD命令行窗口，按下Win+X，选择命令提示符(管理员)。
​	**说明：kms.xspace.in是kms[服务器](http://jump2.bdimg.com/safecheck/index?url=rN3wPs8te/pL4AOY0zAwhz3wi8AXlR5gsMEbyYdIw60/Fd7POtMyKp0TUfuDKnR9ImOuUl9obIdesUqvhcRz+iEQkSXZ+ntRTKCgqz6IivgtjO1hMDXnS43RDmR8I0b+AQVCPRaueoEKs0Fk2ke9LMezOIcW3ZL4GC4kZwiBYFdGuTxvvBVzUBFEYdHJUUewAumNeulcb+D1XcjsXoNPO7gedTYHhfIUyqvBGCnETvQwPGbuJnYGNA==)地址，可能会失效，如果激活失败，可以自行搜索kms服务器地址，将kms.xspace.in替换成新的地址即可，比如换成kms.03k.org**

​	win10专业版用户请依次输入：

~~~
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.xspace.in
slmgr /ato
~~~

