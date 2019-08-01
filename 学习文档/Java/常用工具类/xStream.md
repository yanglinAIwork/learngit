## XStream工具

XStrem 是Java中实体类和xml相互转换的一个插件工具。

---

需要引入的包：

~~~java
<!-- xstream xml和Java对象转化 -->
        <dependency>
            <groupId>xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.1.3</version>
        </dependency>
        <dependency>
            <groupId>xpp3</groupId>
            <artifactId>xpp3</artifactId>
            <version>1.1.3.4.O</version>
        </dependency>
~~~

---

实例：

Team实体类

~~~java
@Data
public class Team {
    private String name;
    private int num;
    private String describe;
}
~~~

Entity2Xml实体类

~~~java
@Data
public class Entity2Xml {
    private Integer money;
    private String firstStr;
    private boolean flag;
}
~~~

测试类Entity2XmlTest

~~~java
public class Entity2XmlTest {

    @org.junit.Test
    public void testName() throws Exception {
        Entity2Xml obj = getEntity();
        XStream stream = new XStream();

        /**
         * 实体类转xml
         */
        String xml = stream.toXML(obj);
        System.out.println(xml);

        /**
         * xml转实体类
         */
        Entity2Xml Entity2Xml = (Entity2Xml) stream.fromXML(xml);
        System.out.println("");
        System.out.println(Entity2Xml.toString());

    }


    public Entity2Xml getEntity(){
        Entity2Xml entity = new Entity2Xml();
        entity.setMoney(30000);
        entity.setFlag(true);
        entity.setFirstStr("艾欧尼亚");

        List<Team> list = new ArrayList<Team>();
        Team team1 = new Team();
        team1.setName("英雄1");
        team1.setNum(2);
        team1.setDescribe("射手英雄，远程攻击型英雄");

        list.add(team1);

        Team team2 = new Team();
        team2.setName("英雄2");
        team2.setNum(3);
        team2.setDescribe("坦克英雄，肉盾抗击打型英雄");

        list.add(team2);

        entity.setTeamList(list);

        return entity;
    }
}
~~~

---

输出结果：

~~~xml
<Entity2Xml>
  <money>30000</money>
  <firstStr>艾欧尼亚</firstStr>
  <flag>true</flag>
  <teamList>
    <Team>
      <name>英雄1</name>
      <num>2</num>
      <describe>射手英雄，远程攻击型英雄</describe>
    </Team>
    <Team>
      <name>英雄2</name>
      <num>3</num>
      <describe>坦克英雄，肉盾抗击打型英雄</describe>
    </Team>
  </teamList>
</Entity2Xml>

Entity2Xml{money=30000, firstStr='艾欧尼亚', flag=true, teamList=[Team{name='英雄1', num=2, describe='射手英雄，远程攻击型英雄'}, Team{name='英雄2', num=3, describe='坦克英雄，肉盾抗击打型英雄'}]}
~~~

