服务提供方 动态地址

```java
@RequestMapping(value ="/comname/{comcode}",method = RequestMethod.GET)
public BaseModel selectComnameByPrimaryKey(@PathVariable String comcode) {

   LdcomMoreBo ldcomMoreBo = ldcomService.selectByPrimaryKey(comcode);
```

服务调用方 动态调用

```java
@RequestMapping(value ="/mechanism/comname/{comcode}",method = RequestMethod.GET)
BaseModel selectByPrimaryKey(@RequestParam("comcode") String comcode);
```

