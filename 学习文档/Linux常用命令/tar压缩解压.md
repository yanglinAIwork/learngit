## tar命令详解

### 一、五个独立命令

| 命令 | 解释                       |
| ---- | -------------------------- |
| -c   | 建立压缩档案               |
| -x   | 解压                       |
| -t   | 查看内容                   |
| -r   | 向压缩归档文件末尾追加文件 |
| -u   | 更新原压缩包中的文件       |

​	**这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用，但是只能用五个中的其中一个。**

---

### 二、参数

​	**-f 是必须的，这个参数是最后一个参数，后面只能接档案名。**

| 参数 | 属性                 |
| ---- | -------------------- |
| -z   | 有gzip属性的         |
| -j   | 有bz2属性的          |
| -Z   | 有compress属性的     |
| -v   | 显示所有过程         |
| -O   | 将文件解开到标准输出 |

---

### 三、示例

#### 1、基础用法

​	**tar -cf all.tar *.jpg**

​	这条命令是将所有的.jpg的文件打成一个名为 all.tar 的包。

​	**tar -rf all.tar *.gif**

​	这条命令是将所有的gif的文件追加到all.tar的包里面去。

​	**tar -uf all.tar logo.gif**

​	这条命令是更新与原来tar包中的logo.gif文件。

​	**tar -tf all.tar**

​	这条命令是列出tar包中所有的文件。

​	**tar -xf all.tar**

​	这条命令是接触all.tar保重所有的文件。

---

#### 2、入门用法

​	**(1)、压缩**

​	**tar -cvf jpg.tar *.jpg (常见)** 

​	将目录里所有的jpg文件打包成jpg.tar。

​	**tar -czf jpg.tar.gz *.jpg (常见)**

​	将目录里所有的jpg文件打包成jpg.tar后，并将其用gzip压缩，生成一个gzip压缩过的包。

​	**tar -cjf jpg.tar.bz2 *.jpg**

​	将目录里所有的jpg文件打包成jpg.tar后，并将其用bzip2压缩，生成一个bzip2压缩过的包。

​	**tar -cZf jpg.tar.Z *.jpg**

​	将目录里所有的jpg文件打包成jpg.tar后，并将其用compress压缩，生成一个umcompress压缩过的包。

​	**rar jpg.rar *.jpg**

​	rar格式压缩，需要先下载rar for linux。

​	**zip jpg.zip *.jpg**

​	zip格式压缩，需要先下载zip for linux。

---

​	**(2)、解压**

​	**tar -xvf file.tar**

​	解压 tar 包

​	**tar -xzvf file.tar.gz**

​	解压tar.gz

​	**tar -xjvf file.tar.bz2**

​	解压 tar.bz2

​	**tar -xZvf file.tar.Z**

​	解压 tar.Z

​	**unrar e file.rar**

​	解压 rar

​	**unzip file.zip** 

​	解压 zip

---

#### 3、中级用法

​	tar -xzvf  zulu8.36.0.1-ca-jdk8.0.202-linux_x64.tar -C /var

​	**-C 是指在指定目录下解压**



### 