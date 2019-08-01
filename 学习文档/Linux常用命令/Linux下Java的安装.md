以 **JDK版本：jdk-6u45-linux-x64.bin** 为例

---

#### 1、安装过程如下：

1) 切换到jdk-6u45-linux-x64.bin文件所在目录。

2) 执行如下命令：

Chmod+x  jdk-6u45-linux-x64.bin 

./jdk-6u45-linux-x64.bin

安装软件会将JDK自动安装到 /usr/java/目录下。 

---

#### 2、配置环境

​	**1) Linux系统**

~~~
#vi /etc/profile

在里面添加如下内容：
export JAVA_HOME=/usr/java/jdk1.6.0_45 
export JAVA_BIN=/usr/java/jdk1.6.0_45/bin
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME JAVA_BIN PATH CLASSPATH
让/etc/profile文件修改后立即生效 ,可以使用如下命令:
# . /etc/profile
注意: . 和 /etc/profile 有空格.
~~~

​	**2) MAC系统**

	touch .bash_profile/open -e .bash_profile
		
	JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home
	PATH=$JAVA_HOME/bin:$PATH:.
	CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
	export JAVA_HOME
	export PATH
	export CLASSPATH
	
	source .bash_profile"使配置生效
在里面添加如下内容：

export JAVA_HOME=/usr/java/jdk1.6.0_45 

export JAVA_BIN=/usr/java/jdk1.6.0_45/bin

export PATH=$PATH:$JAVA_HOME/bin

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export JAVA_HOME JAVA_BIN PATH CLASSPATH

让/etc/profile文件修改后立即生效 ,可以使用如下命令:

\# . /etc/profile

注意: . 和 /etc/profile 有空格.