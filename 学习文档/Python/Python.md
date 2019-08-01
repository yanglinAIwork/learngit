## Python Sftp功能

### 一、安装插件

​	Python Sftp需要用 paramiko相关插件，插件安装方式：

~~~python
pip install paramiko
~~~

---

### 二、代码

	 path="C:/Users/yangLin/Desktop/"
	 fname="zulu8.36.0.1-ca-jdk8.0.202-linux_x64.tar.gz"
	 transport = paramiko.Transport(('10.7.130.30', 22))
	 transport.connect(username='username', password='password!')
	 remote_path="/app/var/"
	 sftp = paramiko.SFTPClient.from_transport(transport)
	 # test.txt 上传至服务器 /app/var/
	 print("本地路径为 %s" % (path))
	 print("传输的文件为 %s" % (fname))
	 os.chdir(path)
	
	sftp.put(fname, os.path.join(remote_path, fname))
	transport.close()
	print("文件传输至%s服务器完成" % (server))
