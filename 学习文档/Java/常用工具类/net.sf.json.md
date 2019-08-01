## NET.SF.JSON

需要的包

~~~
json-lib-2.4-jdk15.jar
commons-lang.jar
commons-beanutils.jar
commons-collections.jar
commons-logging.jar 
~~~

---

### 1、String转JSON

~~~java
JSONObject json = new JSONObject();
String str = "{\"resultcontent\":\"哈哈哈哈\",\"resultcode\":\"0\"}";
json = JSONObject.fromObject(str);
System.out.println(json);
~~~

---

### 2、Json转String

~~~java
JSONObject json = new JSONObject();
String string = json.toString();
System.out.println(string);
~~~

---

### 3、Map转JSON

```java
JSONObject json = new JSONObject();
Map<String, String> map = new HashMap<>();
map.put("resultcode","0");
map.put("resultcontent","哈哈哈哈");
json = JSONObject.fromObject(map);
System.out.println(json);

//{"resultcontent":"哈哈哈哈","resultcode":"0"}
```

---

### 4、Json转Map

~~~java
JSONObject json = new JSONObject();
Map<String, String> map = new HashMap<>();
map = (Map)json;
System.out.println(map);

//{"resultcontent":"哈哈哈哈","resultcode":"0"} 原本格式{resultcontent=哈哈哈哈, resultcode=0}，但是能get到
~~~

---

### 5、List转Json

~~~java
 List<String> stringList = new ArrayList<>();
 stringList.add("1");
 stringList.add("2");
 JSONArray jsonArray = JSONArray.fromObject(stringList);
 System.out.println(jsonArray);

//["1","2"]
~~~

---

### 6、Json转List

~~~java
 String jsonStr = "{\"body\":{\"insuredinfo\":[{\"beInsuredName\":\"622722197901152111\",\"sex\":\"0\",\"customId\":\"100005487710\",\"idNo\":\"段斌\"},{\"beInsuredName\":\"622722197901152411\",\"sex\":\"0\",\"customId\":\"100005487728\",\"idNo\":\"段小斌\"}]}}";

JSONObject json = JSONObject.fromObject(jsonStr);

List<BeInsuredBean> beInsuredBeanList = new ArrayList<>();
List<BeInsuredBean> beInsuredBeanList2 = new ArrayList<>();
JSONArray jsonArray = json.getJSONObject("body").getJSONArray("insuredinfo");
//System.out.println(jsonArray);
//方式一：这个把list格式转了但是能用
beInsuredBeanList = JSONArray.toList(jsonArray,new BeInsuredBean(),new JsonConfig());
System.out.println(beInsuredBeanList);
//[BeInsuredBean{beInsuredName='622722197901152111', idNo='段斌', customId='100005487710', sex='0'}, BeInsuredBean{beInsuredName='622722197901152411', idNo='段小斌', customId='100005487728', sex='0'}]

//方式二：这个是正常的list格式
for(int i = 0;i < jsonArray.size(); i++) {
    JSONObject jsonObject = jsonArray.getJSONObject(i);
    BeInsuredBean beInsuredBean = (BeInsuredBean) JSONObject.toBean(jsonObject,BeInsuredBean.class);
    beInsuredBeanList2.add(beInsuredBean);
}
System.out.println(beInsuredBeanList2);
//[BeInsuredBean{beInsuredName='622722197901152111', idNo='段斌', customId='100005487710', sex='0'}, BeInsuredBean{beInsuredName='622722197901152411', idNo='段小斌', customId='100005487728', sex='0'}]
~~~

---

### 7、XML与JSON之间的转换

需要导入xom-1.1.jar

~~~java
public class JSONArrayToArray {
    public static void main(String[] args) {
        //Java数组转换JSONArray
        boolean[] boolArray = new boolean[] {true, false, true};
        JSONArray jsonArray = JSONArray.fromObject(boolArray);
        System.out.println("1："+ jsonArray.toString());

        //JSONArray转换Java数组
        Object obj[] = jsonArray.toArray();
        for (Object o : obj) {
        System.out.print("2：" + o + " ");
        }
    }
}

结果：
1：[true,false,true]
2：true 2：false 2：true 
~~~

---

### 8、Json和Bean互转

~~~java
//json转实体类
JSONObject jsonObject = new JSONObject();
BeInsuredBean beInsuredBean = 	  (BeInsuredBean)JSONObject.toBean(jsonObject,BeInsuredBean.class);

//实体类转json
JSONObject jsonBeInsuredBean = JSONObject.fromObject(beInsuredBean);  
~~~

---

### 9、有个坑

​	json强转为object之后，如果有null值（例如a=,b=），此时直接把object类返回给前端时，http会默认转为json对象，但由于a=,b=不是json标准的a="",b=""，就回报异常。

​	解决方法：

​	在return给页面之前，把object json格式化一下即可。< JSONObject.fromObject(Object) >

---

## 附件

### BeInsuredBean实体类

~~~java
package com.slup.customer.model;

import java.io.Serializable;

/**
 * @program: slup
 * @Date: 2019/4/29 10:35
 * @Author: Mr.Yang
 * @Description: 被保人信息实体类
 */
public class BeInsuredBean implements Serializable {
    private static final long serialVersionUID = 1L;

    /**
     * 被保人姓名
     */
    private String beInsuredName;
    /**
     * 证件号
     */
    private String idNo;

    /**
     * 客户号
     */
    private String customId;
    /**
     * 客户性别
     */
    private String sex;

    public String getCustomId() {
        return customId;
    }

    public void setCustomId(String customId) {
        this.customId = customId;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getBeInsuredName() {
        return beInsuredName;
    }

    public void setBeInsuredName(String beInsuredName) {
        this.beInsuredName = beInsuredName;
    }

    public String getIdNo() {
        return idNo;
    }

    public void setIdNo(String idNo) {
        this.idNo = idNo;
    }

    public BeInsuredBean(String beInsuredName, String idNo, String customId, String sex) {
        this.beInsuredName = beInsuredName;
        this.idNo = idNo;
        this.customId = customId;
        this.sex = sex;
    }

    public BeInsuredBean() {
    }

    @Override
    public String toString() {
        return "BeInsuredBean{" +
                "beInsuredName='" + beInsuredName + '\'' +
                ", idNo='" + idNo + '\'' +
                ", customId='" + customId + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}

~~~





```
public static String doPost(String url, Map<String, String> headers, Map<String, String> params) throws Exception {
   // 创建httpClient对象
   CloseableHttpClient httpClient = HttpClients.createDefault();

   // 创建http对象
   HttpPost httpPost = new HttpPost(url);
   /**
    * setConnectTimeout：设置连接超时时间，单位毫秒。
    * setConnectionRequestTimeout：设置从connect Manager(连接池)获取Connection
    * 超时时间，单位毫秒。这个属性是新加的属性，因为目前版本是可以共享连接池的。
    * setSocketTimeout：请求获取数据的超时时间(即响应时间)，单位毫秒。
    * 如果访问一个接口，多少时间内无法返回数据，就直接放弃此次调用。
    */
   RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(CONNECT_TIMEOUT)
         .setSocketTimeout(SOCKET_TIMEOUT).build();
   httpPost.setConfig(requestConfig);
   // 设置请求头
   /*
    * httpPost.setHeader("Cookie", ""); httpPost.setHeader("Connection",
    * "keep-alive"); httpPost.setHeader("Accept", "application/json");
    * httpPost.setHeader("Accept-Language", "zh-CN,zh;q=0.9");
    * httpPost.setHeader("Accept-Encoding", "gzip, deflate, br");
    * httpPost.setHeader("User-Agent",
    * "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
    * );
    */
   packageHeader(headers, httpPost);

   // 封装请求参数
   packageParam(params, httpPost);

   // 创建httpResponse对象
   CloseableHttpResponse httpResponse = null;

   try {
      // 执行请求并获得响应结果
      return getHttpClientResult(httpResponse, httpClient, httpPost);
   } finally {
      // 释放资源
      release(httpResponse, httpClient);
   }
}
```

### 