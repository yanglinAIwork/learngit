### 一、需要导包

```java
import java.io.File;
```

---

### 二、常见用法

#### 1、获取当前class类路径

```java
try {
    File path = new File(ResourceUtils.getURL("classpath:").getPath());
    if (!path.exists()) {
        path = new File("");
    }
    System.out.println(path.toString());
} catch (Exception e){
    e.printStackTrace();
}
```

输出结果：

~~~
E:\SpringBoot_2.0.3\finchley\consul-service\target\classes
~~~

---

#### 2、创建文件

​	不完美代码，有优化的在下面

~~~java
import java.io.File ;
import java.io.IOException ;
public class FileDemo01{
    public static void main(String args[]){
        File f = new File("d:\\test.txt") ;        // 实例化File类的对象，给出路径
        try{
            f.createNewFile() ;        // 创建文件，根据给定的路径创建
        }catch(IOException e){
            e.printStackTrace() ;    // 输出异常信息
        }
    }
};
~~~

//以上完成了文件创建功能，但是开发中以上程序编写肯定会出现错误，在各个操作系统中，路径分隔符是不一样的。例如：
//windows使用反斜杠："\"
//Linux中使用正斜杠："/"
//要想让JAVA可移植增强，最好让操作系统自动使用分隔符。

~~~java
System.out.println("separator：" + File.separator) ;    // 调用静态常量
//	separator：\
~~~

优化创建文件

~~~java
File f = new File("d:"+File.separator+"test.txt") ;        // 实例化File类的对象
try{
	f.createNewFile() ;        // 创建文件，根据给定的路径创建
}catch(IOException e){
	e.printStackTrace() ;    // 输出异常信息
}
~~~

---

#### 3、获取文件路径

~~~java
File file = new File("");
System.out.println(file.getAbsolutePath());
System.out.println(file.getCanonicalPath());
//E:\iDealWorkspace\slup_R201905020A_prod20190429_yanglin\pic
File file = new File(ResourceUtils.getURL("classpath:").getPath());
System.out.println(file.toString());
////E:\iDealWorkspace\slup_R201905020A_prod20190429_yanglin\class+???路径
~~~

#### 4、删除文件夹

```java
private boolean deleteFile(File dirFile) {
    // 如果dir对应的文件不存在，则退出
    if (!dirFile.exists()) {
        return false;
    }

    if (dirFile.isFile()) {
        return dirFile.delete();
    } else {

        for (File file : dirFile.listFiles()) {
            deleteFile(file);
        }
    }

    return dirFile.delete();
}

mian(){
    File fileTmp = new File(picur);
    if(fileTmp.isDirectory()){
        deleteFile(fileTmp);
    }
}
```

#### 5、base64转图片

~~~java
private static void inputStreamToFile(InputStream ins,File file) {
    try {
        OutputStream os = new FileOutputStream(file);
        int bytesRead = 0;
        byte[] buffer = new byte[8192];
        while ((bytesRead = ins.read(buffer, 0, 8192)) != -1) {
            os.write(buffer, 0, bytesRead);
        }
        os.close();
        ins.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

main(){
    byte[] bytes = decoder.decodeBuffer(stringBase64);
    InputStream input = new ByteArrayInputStream(bytes);
    file = new File("C:" + File.separator + "tupian.jpg");
    //生成图片到本地成功
    inputStreamToFile(input, file);
}
~~~



---

### 三、api

![img](https://images2015.cnblogs.com/blog/779030/201608/779030-20160807145748247-185833785.png)

