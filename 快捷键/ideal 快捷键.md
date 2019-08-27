## 一、快捷键篇

1. Shift + F10 快速run

2. Shift + F9  快速DeBug，F8下一步

3. alt + enter  快速补全  相当于 eclipse 的alt +/

4. Ctrl + Alt + **方法** 可以查看实现类

5. sout 快速 system.out....

6. 快速main方法 psvm

7. System.out.println(String.format("s:%s", s));

8. Ctrl + r 替换全局的变量

9. ctrl + shift + r 替换全部文本的变量、

10. Ctrl + Alt + L 快速格式化

11. /*mould 方法快速注释

12. Ctrl + Shift + T 快速生成测试类

    注意事项：这边要注意下[IntelliJ IDEA 的 Content Root](https://blog.csdn.net/zx48822821/article/details/78640041)

    - 如果没有设置测试源根，则会造成我上述的问题，生成的test文件和文件会在同一文件夹下。

      > ![IDEA 自动生成Junit进行单元测试](https://s1.51cto.com/images/blog/201806/29/ecd21f912eefe871099665cdf8e8334e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    - 通过配置Content Root 【新增一个test文件夹在根目录下，并点击Mark as - Tests按钮，完成Add Content Root】

      > ![IDEA 自动生成Junit进行单元测试](https://s1.51cto.com/images/blog/201806/29/135adbbe170669fe993fd74390bacc77.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    #### 最后效果如下

    > ![IDEA 自动生成Junit进行单元测试](https://s1.51cto.com/images/blog/201806/29/095e526113cc465e8db8f18aed639128.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

13. 字符串转换大小写 alt +m



## 二、快捷设置篇

1.  在最下面一行中有Version Control 的第二个选项可以查看本地修改过的代码
2.  打开setting -> Editor -> File Type 选择最下面一行ignore，在末尾添加不想要显示的文件 ，例如: *.iml

---

## 三、有目录篇

### 1、IDEA提示不区分大小写

​		Alt + Ctrl + s 弹出Settings设置，输入`sensitive`，将`case sensitive completion`的值改为NONE

### 2、idea的环境配置默认保存位置

​		idea的环境默认保存的位置`C:\Users\xxxxxxxxx\.IntelliJIdea14` ，xxxxxx代表用户目录，可以对该目录进行备份，一旦环境出问题恢复此配置即可。或者可以在`%IDEA_HOME%/bin/idea.properties`中修改该配置路径。

### 3、隐藏不想看到的文件或者文件夹

​		Alt + Ctrl + s 弹出Settings设置，搜索`File Type`，在`Ignore files and folders`添加需要隐藏的文件(或后缀)。

### 4、显示行号

​		Alt + Ctrl + s 弹出Settings设置，

​		选中`Editor ` -> `General ` -> `Appearance ` -> `Show line numbers`

### 5、改变字体大小

​		Alt + Ctrl + s 弹出Settings，在`Editor`的`FONTS`中`Size`修改大小。

### 6、快捷输出

```java
System.out.println 快捷输出
“abc”.sout => System.out.println("abc");
```

### 7、For循环快捷生成

```java
for循环
List<String> list = new ArrayList<String>();
输入: list.for 即可输出
for(String s:list){} 
```

### 8、实现Serializable接口

​		Alt + Ctrl + S 弹出Settings，在`Editor`中选`nspections`，在新的窗口搜索`Serialization issues`，选`Serializable class without ’serialVersionUID`，即可。

### 9、演出模式<即当前页代码最大化>

​		Alt + V，选择`Enter Presentation Mode`

​		退出演出模式：同样操作

