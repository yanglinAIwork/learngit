```java
List<String> comcodeList = (List<String>) redisUtil.get("comcodeAll");
for(String com:comcodeList){
   JsonObject c = new JsonParser().parse(com).getAsJsonObject();
   String comcode = c.get("comcode").getAsString();
   // 获取对应comcode的代理机构名称
   if(agentInfo.getManagecom().equals(comcode)){
      String name = c.get("name").getAsString();
      if(!"".equals(name)&&name!=null){
         agentInfo.setManageComName(name);
         getflag = true;
         break;
      }
   }
}
```

```
//1、使用JSONObject
JSONObject jsonObject = JSONObject.fromObject(tuser);
UserOtherBo userBo = (UserOtherBo) JSONObject.toBean(jsonObject, UserOtherBo.class);
```



```
public static Object convertJsonToObj(String json, Class c){
   if(json == null){
      return null;
   }else{
      ObjectMapper mapper = new ObjectMapper();
      mapper.setDateFormat(getDateFormat());
      Object object = null;
      try {
         object = mapper.readValue(json, c);
      } catch (Exception e) {
         e.printStackTrace();
      }
      return object;
   }
}
```

```
StringEntity entity = new StringEntity(json, ContentType.APPLICATION_JSON);

public static String doGet(String url, Map<String, String> headers, Map<String, String> params) throws Exception {
   // 创建httpClient对象
   CloseableHttpClient httpClient = HttpClients.createDefault();

   // 创建访问的地址
   URIBuilder uriBuilder = new URIBuilder(url);
   if (params != null) {
      Set<Entry<String, String>> entrySet = params.entrySet();
      for (Entry<String, String> entry : entrySet) {
         uriBuilder.setParameter(entry.getKey(), entry.getValue());
      }
   }

   // 创建http对象
   HttpGet httpGet = new HttpGet(uriBuilder.build());
   /**
    * setConnectTimeout：设置连接超时时间，单位毫秒。
    * setConnectionRequestTimeout：设置从connect Manager(连接池)获取Connection
    * 超时时间，单位毫秒。这个属性是新加的属性，因为目前版本是可以共享连接池的。
    * setSocketTimeout：请求获取数据的超时时间(即响应时间)，单位毫秒。
    * 如果访问一个接口，多少时间内无法返回数据，就直接放弃此次调用。
    */
   RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(CONNECT_TIMEOUT)
         .setSocketTimeout(SOCKET_TIMEOUT).build();
   httpGet.setConfig(requestConfig);

   // 设置请求头
   packageHeader(headers, httpGet);

   // 创建httpResponse对象
   CloseableHttpResponse httpResponse = null;

   try {
      // 执行请求并获得响应结果
      return getHttpClientResult(httpResponse, httpClient, httpGet);
   } finally {
      // 释放资源
      release(httpResponse, httpClient);
   }
}
```