### Lambad表达式由来：

​	**Lambda 是一个匿名函数**，我们可以把 Lambda 表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。

​	很长一段时间java被吐槽是冗余和缺乏**函数式编程能力**的语言，随着函数式编程的流行java8也引入了 这种编程风格。在此之前我们都在写匿名内部类干这些事，但有时候这不是好的做法。

---

### 1、无参写法

~~~java
public static void main(String[] args) {

        /**
         * 原写法
         */
        Runnable run = new Runnable() {
            @Override
            public void run() {
                System.out.println("haha");
            }
        };
        run.run();

        /**
         * Lambda引入之后,把匿名内部类写法改为 ()->，即把new Runnable() {} 全部替换为 ()->
         */
        Runnable runNew = () -> System.out.println("哈哈哈");
        runNew.run();
    }
~~~

​	用()和->的方式完成了这件事，这是一个没有名字的函数。使用->将参数和实现逻辑分离，当运行这个线程的时候执行的是->之后的代码片段，且编译器帮助我们做了类型推导； 这个代码片段可以是用{}包含的一段逻辑。

---

### 2、有参写法

~~~java
/**
 * 原写法
 */
TreeSet<String> treeSet = new TreeSet<>(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                //(x < y) ? -1 : ((x == y) ? 0 : 1);
                return Integer.compare(o1.length(),o2.length());
            }
        });
treeSet.add("hhh");
treeSet.add("aaa");
treeSet.add("kkkk");
treeSet.add("kk");

//Integer.compare如果两个字符长度一致只保留第一个。否则按长度小-大排序

/**
 * 引入lambda表达式之后，传的参数不需要定义类型
 */
TreeSet<String> testDemo = new TreeSet<>((o1,o2)->Integer.compare(o1.length(),o2.length()));
~~~

---

3、

~~~java
	@Test
	public void test1(){
		TeacherService t = new TeacherService();
		List<Teacher> all = t.filter(e->true);
		for (Teacher teacher : all) {
			System.out.println(teacher);
		}
		
		List<Teacher> all2 = t.filter(e->e.getAge()>30);
		for (Teacher teacher : all2) {
			System.out.println(teacher);
		}
		
		List<Teacher> all3 = t.filter(e->e.getSalary()>5000);
		for (Teacher teacher : all3) {
			System.out.println(teacher);
		}
	}
~~~

