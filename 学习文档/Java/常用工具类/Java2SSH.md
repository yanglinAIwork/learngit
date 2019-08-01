# JAVA远程(ssh)执行linux脚本

### 1、导入maven依赖

~~~xml
		<!-- java远程ssh调用sh的jar包 Start -->
        <dependency>
            <groupId>org.jvnet.hudson</groupId>
            <artifactId>ganymed-ssh2</artifactId>
            <version>build210-hudson-1</version>
        </dependency>
        <!-- java远程ssh调用sh的jar包 End -->

        <!-- IO流包 Start -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.5</version>
        </dependency>
        <!-- IO流包 End -->
~~~

---

### 2、工具类代码

~~~java
import ch.ethz.ssh2.ChannelCondition;
import ch.ethz.ssh2.Connection;
import ch.ethz.ssh2.Session;
import ch.ethz.ssh2.StreamGobbler;
import org.apache.commons.io.IOUtils;

import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.nio.charset.Charset;

/**
 * @program: study
 * @Date: 2019/4/24 10:05
 * @Author: Mr.Yang
 * @Description: Java 远程SSH调用 sh脚本
 */
public class Java2SSH {

    private Connection connection;

    /**
     * 远程ip
     */
    private String ip;

    /**
     * 用户名/密码
     */
    private String userName;
    private String passWord;

    /**
     * 字符集编码
     */
    private String charset = Charset.defaultCharset().toString();

    /**
     * 超时时间
     */
    private static final int TIMEOUT = 1000*5*60;

    /**
     * 构造函数
     * @param userName
     * @param passWord
     * @param ip
     */
    public Java2SSH(String userName, String passWord, String ip){
        this.userName = userName;
        this.passWord = passWord;
        this.ip = ip;
    }

    /**
     * 登录方法
     * @return
     * @throws IOException
     */
    private boolean login() throws IOException {
        connection = new Connection(ip);
        connection.connect();
        /*返回认证结果*/
        return connection.authenticateWithPassword(userName, passWord);
    }

    /**
     * 获取流通道数据
     * @param inputStream
     * @param charset
     * @return
     * @throws Exception
     */
    private String processStream(InputStream inputStream, String charset)
            throws Exception{
        byte[] buf = new byte[1024];
        StringBuilder sb = new StringBuilder();
        while(inputStream.read(buf) != -1){
            sb.append(new String(buf, charset));
        }
        return sb.toString();
    }

    /**
     *
     */
    private int exec(String cmd) throws Exception{
        InputStream outStream = null;
        InputStream errStream = null;

        String outStr = "";
        String errStr = "";

        int ret = -1;

        try {
            if(login()){
                /*打开session会话通道*/
                Session session = connection.openSession();
                /**
                 * 执行cmd命令
                 * 用方法execCommand执行Shell命令的时候，会遇到获取不全环境变量的问题，
                 * 可能是因为session刚刚建立还没有读取完默认信息的时候，execCommand就执行了Shell命令
                 *
                 */
                /*session.execCommand(cmd);*/
                /*解决方法*/
                /*建立虚拟终端*/
                session.requestPTY("bash");
                /*打开一个shell*/
                session.startShell();
                /*准备输入命令*/
                PrintWriter out = new PrintWriter(session.getStdin());
                /*输入执行命令*/
                out.println(cmd);
                out.println("exit");
                /*关闭输入流*/
                out.close();
                /*解决方法 End*/

                /*获得输出结果*/
                outStream = new StreamGobbler(session.getStdout());
                outStr = processStream(outStream, charset);
                /*获得异常信息*/
                errStream = new StreamGobbler(session.getStderr());
                errStr = processStream(errStream, charset);
                /* 等待，除非1.连接关闭；2.输出数据传送完毕；3.进程状态为退出；4.超时,此处用到的是 3和4*/
                session.waitForCondition(ChannelCondition.EXIT_STATUS, TIMEOUT);

                /*System.out.println("输出结果: " + outStr);*/
                /*System.out.println("异常信息: " + errStr);*/

                /*获取结果*/
                ret = session.getExitStatus();
            }

        } catch (Exception e){
            throw new Exception("登录远程机器失败" + ip);
        } finally {
            if(connection != null){
                connection.close();
            }
            IOUtils.closeQuietly(outStream);
            IOUtils.closeQuietly(errStream);
        }
        return ret;
    }

    public static void main(String[] args) throws Exception {
        String ip = "10.7.130.31";
        String username = "appuser";
        String passWord = "DVk6hJD!";
        Java2SSH java2SSH = new Java2SSH(username,passWord,ip);
        int exec = java2SSH.exec("sh /app/slup-servers/test.sh");
        System.out.println(exec);
    }

}

~~~

---

### 3、注意事项

​	当一个Shell命令执行时间过长时，会遇到ssh连接超时的问题，

解决办法：

1. 之前通过把Linux主机的sshd_config的参数ClientAliveInterval设为60，同时将第7步中timeout时间设置很大，来保证命令执行完毕，因为是执行Mahout中一个聚类算法，耗时最少7、8分钟，数据量大的话，需要几个小时。

2. 将命令改成了nohup的方式执行，nohup hadoop jar .... >> XXX.log && touch XXX.log.end &

   这种方式是提交到后台执行，即使当前连接断开也会继续执行，把命令的输出结果写入日志，如果hadoop命令执行成功，则生成.end文件

    获取文件的方法 ganymed-ssh2-build210.jar 也提供了，如下

   ~~~java
   SCPClient scpClient = con.createSCPClient();  
   scpClient.get("remoteFiles","localDirectory");  //从远程获取文件　　　　　　　　　　　　
   ~~~

