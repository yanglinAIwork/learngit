#### Dockerfile

在同一目录下，创建Dockerfile

```
# vi Dockerfile 
FROM python:2.7
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD python app.py
```

对上面的Dockerfile做一下简单说明：

- 容器使用Python 2.7的镜像
- 将当前目录下文件拷贝到容器内/code
- 指定工作目录为/code
- 安装python需要的库：flask, redis
- 容器执行命令 python app.py

