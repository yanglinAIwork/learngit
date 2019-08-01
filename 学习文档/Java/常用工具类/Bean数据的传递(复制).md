## Java中Bean实体类值的传递(复制)

这篇讲解三种方法：BeanUtils、PropertyUtils、Dozer

### 一、BeanUtils

#### 1、pom依赖

~~~xml
<dependency>
	<groupId>commons-beanutils</groupId>
	<artifactId>commons-beanutils</artifactId>
</dependency>
~~~

#### 2、实体类复制示例<PO转换>

~~~java
package com.example.demo.demo;

import org.springframework.beans.BeanUtils;

/**
 * @program: demo
 * @Date: 2019/7/9 11:09
 * @Author: Mr.Yang
 * @Description:
 */
public class CompareBeanUtils {
    public static void main(String[] args) {
        MessageBo m = new MessageBo("1270000001","12345678",
                "模拟","模拟数据","2019-07-09",
                "模拟用户","这是一个模拟数据");
        System.out.println(m.toString());

        /** 用BeanUtils将值传递到另一个实体类 */
        MessageBoCopy messageBoCopy = new MessageBoCopy();
        BeanUtils.copyProperties(m, messageBoCopy);
        System.out.println(messageBoCopy.toString());
    }
}

//结果
MessageBo{agentCode='1270000001', contno='12345678', customerNo='模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
MessageBoCopy{agentCode='1270000001', contno='12345678', customerNo='模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
~~~

#### 3、参数传递

~~~java
try {
    /** 修改实体类中原属性的值，没有属性值不会报错 */
	BeanUtils.setProperty(m,"customerNo","修改后的模拟");
	System.out.println(m.toString());
    
    /** 获取实体类中属性的值，没有属性值会报错 -NoSuchMethodException */
    BeanUtils.getProperty(m,"customerNo");
    
    /** 转map */
    Map<String, String> map = new HashMap<String, String>();
    map = BeanUtils.describe(m);
    System.out.println("-----------------------------\r\n" + BeanUtils.describe(m));
    
    /** map赋值 */
    BeanUtils.setProperty(map, "customerNo","No");
    System.out.println(map);
    
} catch (IllegalAccessException e) {
	e.printStackTrace();
} catch (InvocationTargetException e) {
	e.printStackTrace();
}

/** 结果
MessageBo{agentCode='1270000001', contno='12345678', customerNo='修改后的模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
-----------------------------
{mainRiskName=模拟数据, agentCode=1270000001, description=这是一个模拟数据, contno=12345678, signDate=2019-07-09, class=class com.example.demo.demo.MessageBo, customerName=模拟用户, customerNo=模拟}
{mainRiskName=模拟数据, agentCode=1270000001, description=这是一个模拟数据, contno=12345678, signDate=2019-07-09, class=class com.example.demo.demo.MessageBo, customerName=模拟用户, customerNo=No}
*/ 
~~~

---

### 二、PropertyUtils

#### 1、pom <同BeanUtils>

~~~xml
<dependency>
	<groupId>commons-beanutils</groupId>
	<artifactId>commons-beanutils</artifactId>
</dependency>
~~~

#### 2、实体类复制示例<PO转换>

~~~java
package com.example.demo.demo;

import org.apache.commons.beanutils.PropertyUtils;
import java.lang.reflect.InvocationTargetException;

/**
 * @program: demo
 * @Date: 2019/7/9 11:09
 * @Author: Mr.Yang
 * @Description:
 */
public class CompareBeanUtils {
    public static void main(String[] args) {
        MessageBo m = new MessageBo("1270000001","12345678",
                "模拟","模拟数据","2019-07-09",
                "模拟用户","这是一个模拟数据");
        System.out.println(m.toString());

        /** 用BeanUtils将值传递到另一个实体类 */
        MessageBoCopy messageBoCopy = new MessageBoCopy();
        try {
            PropertyUtils.copyProperties(messageBoCopy,m);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
        System.out.println(messageBoCopy.toString());
    }
}


//结果
MessageBo{agentCode='1270000001', contno='12345678', customerNo='模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
MessageBoCopy{agentCode='1270000001', contno='12345678', customerNo='模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
~~~

#### 3、参数传递

~~~java
try {
    /** 修改实体类中原属性的值，没有属性值不会报错 */
	PropertyUtils.setProperty(m,"customerNo","修改后的模拟");
	System.out.println(m.toString());
    
    /** 获取实体类中属性的值，没有属性值会报错 -NoSuchMethodException */
    PropertyUtils.getProperty(m,"customerNo");
    
    /** PO转Map */
    Map<String, Object> map = new HashMap<String, Object>();
    map = PropertyUtils.describe(m);
    System.out.println("-----------------------------\r\n" + map);
    
    /** map赋值 */
    BeanUtils.setProperty(map, "customerNo","No");
    System.out.println(map);
    
} catch (IllegalAccessException e) {
	e.printStackTrace();
} catch (InvocationTargetException e) {
	e.printStackTrace();
}

/** 结果
MessageBo{agentCode='1270000001', contno='12345678', customerNo='修改后的模拟', mainRiskName='模拟数据', signDate='2019-07-09', customerName='模拟用户', description='这是一个模拟数据'}
-----------------------------
{mainRiskName=模拟数据, agentCode=1270000001, description=这是一个模拟数据, contno=12345678, signDate=2019-07-09, class=class com.example.demo.demo.MessageBo, customerName=模拟用户, customerNo=模拟}
{mainRiskName=模拟数据, agentCode=1270000001, description=这是一个模拟数据, contno=12345678, signDate=2019-07-09, class=class com.example.demo.demo.MessageBo, customerName=模拟用户, customerNo=No}
*/ 
~~~

---

### 三、Dozer

​		详细介绍 ：地址 Dozer Demo Github

#### 1、pom

~~~xml
<dependency>
    <groupId>com.github.dozermapper</groupId>
    <artifactId>dozer-spring-boot-starter</artifactId>
    <version>6.2.0</version>
</dependency>
~~~

#### 2、PO转换示例

~~~java
@Autowired
private Mapper dozerMapper;

……
StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
log.info("StudentVo: [{}]", studentVo.toString());
……

~~~

#### 3、属性名不同时转换

~~~java
// 地址
private String address;

---
// 地址
private String addr;

---
@Override
protected void configure() {
    //测试所有properties，为不同名的 property 手动配置映射关系
    mapping(StudentDomain.class, StudentVo.class)
        .fields("address", "addr");
}

---
StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
log.info("StudentVo: [{}]", studentVo.toString());
~~~

#### 4、关闭隐式配置

​		Dozer 默认是隐式匹配，如果我们关闭隐士匹配，Dozer 只会为我们匹配我们显式指定的 field修改 configure

~~~java
//关闭隐式匹配
mapping(StudentDomain.class, StudentVo.class, TypeMappingOptions.wildcard(false))
    .fields("address", "addr");

//重新运行 用例1的测试方法 ，运行结果(只有地址做了映射)：
//StudentVo: [StudentVo(id=null, name=null, age=null, mobile=null, addr=中国)]
~~~

#### 5、排除不想转换的字段

​		默认我们要使用 Dozer 的隐式匹配（同名字段全部匹配），但我们不想将学生的 mobile 字段做映射，我们可以通过 `exclude` 方法排除不想映射的字段修改 configure

~~~java
//测试所有properties，为不同名的 property 手动配置映射关系，排除 mobile 字段
mapping(StudentDomain.class, StudentVo.class)
        .exclude("mobile")
        .fields("address", "addr");

//重新运行 用例1的测试方法 ，运行结果：
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=null, addr=中国)]
~~~

#### 6、对象内嵌套/递归对象

​		对象通常嵌套对象或者集合对象，Dozer 可以递归完成相关映射将学生地址封装，同时为学生添加多门课程新增 AddressDomain.java 和 AdressVo.java，除详细地址外所有字段相同

​		**Dozer 会隐式递归匹配所有 field，甚至集合**

~~~java
@Data
public class AddressDomain {
    // 省
    private String province;
    // 市
    private String city;
    // 区
    private String district;
    // 详细
    private String detail;
}
~~~

~~~java
@Data
public class AddressVo {
    ... 省略省市区，同 AddressDomain
    // 详细
    private String detailAddr;
}
~~~

同时创建课程类 CourseDomian.java 和 CourseVo.java, 内容相同

~~~java
@Data
public class CourseDomain {
    // 课程编码
    private String courseCode;
    // 课程Id
    private Integer courseId;
    // 课程名称
    private String courseName;
    // 老师名称
    private String teacherName;
}
~~~

同时修改 StudentDomain.java 和 StudentVo.java, 添加地址和课程集合字段：

~~~java
// 地址
private AddressDomain address;
// 课程集合
private List<CourseDomain> courses;
~~~

修改configure 配置

~~~java
mapping(AddressDomain.class, AddressVo.class)
    .fields("detail", "detailAddr");
~~~

测试用例

~~~java
@Test
public void testCascadeObject(){
    StudentDomain studentDomain = getStudentDomain();

    StudentVo studentVo = dozerMapper.map(studentDomain, StudentVo.class);
    log.info("StudentVo: [{}]", studentVo.toString());
}

/** 结果
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)])]
*/
~~~

#### 7、属性类型不同

​		修改 StudentDomain.java 的 age 字段为 Integer 类型，修改 StudentVo.java 的 age 字段为 String 类型重新运行上述测试用例，双向映射，一切正常结论：Dozer 开箱即用的功能之一就是类型转换，**多数**类型我们不需要手动转换类型，完全交给 Dozer即可。

​		注意：但是 Date 和 String 不可以，需要指定 `date-formate` 格式为学生添加入学日期 entrollmentDate，在 StudentDomain.java 中是 String 类型，在 StudentVo.java 中是 `java.util.Date` 类型

~~~java
// 入学日期 设置为"2019-09-01 10:00:00"
private String entrollmentDate;

---
//为 entrollmentDate 字段配置  date-formate ，修改 configure
mapping(StudentDomain.class, StudentVo.class TypeMappingOptions.dateFormat("yyyy-MM-dd"))
    .fields("courses[0].teacherName", "counsellor");
/* 运行结果：
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)], counsellor=京晶, entrollmentDate=Sun Sep 01 00:00:00 CST 2019)]
*/
~~~



#### 8、深度匹配需求

​		可以通过深度匹配指定字段，匹配方式以 "." 号进行分割，集合属性可以指定索引

~~~java
//测试深度索引匹配
mapping(StudentDomain.class, StudentVo.class)
        .fields("courses[0].teacherName", "counsellor");

//重新运行测试用例，运行结果： 多了个counsellor=京晶
StudentVo: [StudentVo(id=1024, name=tan日拱一兵, age=18, mobile=13996996996, address=AddressVo(province=北京, city=北京, district=海淀区, detailAddr=西二旗), courses=[CourseVo(courseCode=English, courseId=1, courseName=英语, teacherName=京晶), CourseVo(courseCode=Chinese, courseId=2, courseName=语文, teacherName=水寒)], counsellor=京晶)]
~~~



---

### 比较：

1、万次内的效率beanutils比dozer要快

2、十万次以上的效率beanutils的效率和dozer差不多。

3、以上是在只考虑基本类型的情况下比较的，dozer如果深拷贝或者关联拷贝时，效率应该会更慢。

4、PropertyUtils和BeanUtils的功能基本一致，唯一的区别是：BeanUtils在对Bean赋值时会进行类型转化，而PropertyUtils不会对类型进行转化，如果类型不同则会抛出异常！，这可以解释PropertyUtils效率比其他两种要高的原因。

建议：基本类型在源目标类型一致的情况下使用： PropertyUtils效率会更高。    复杂类型的拷贝可以使用： Dozer。



## 附录：

JavaBean实体类 <另一个实体类结构与之相同>

~~~java
package com.example.demo.demo;

import java.io.Serializable;
@Data
public class MessageBo implements Serializable{

	private static final long serialVersionUID = 1643076599388122406L;
	
	private String agentCode;
	private String contno;
	private String customerNo;
	private String mainRiskName;
	private String signDate;
	private String customerName;
	private String description;
}

~~~

