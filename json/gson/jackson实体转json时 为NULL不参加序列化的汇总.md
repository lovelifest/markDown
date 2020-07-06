# [jackson实体转json时 为NULL不参加序列化的汇总](https://www.cnblogs.com/weiapro/p/7653443.html)

首先加入依赖

```
<dependency>
<groupId>com.google.code.gson</groupId>
<artifactId>gson</artifactId>
</dependency>
```

方法一、实体上使用 @JsonInclude(JsonInclude.Include.NON_NULL)

> 1、如果放在属性上，如果该属性为NULL则不参与序列化 ;
> 2、如果放在类上，那对这个类的全部属性起作用 ;

> 参数意义：
>
> ```java
> JsonInclude.Include.ALWAYS       默认
> 
> JsonInclude.Include.NON_DEFAULT   属性为默认值不序列化
> 
> JsonInclude.Include.NON_EMPTY     属性为 空（””） 或者为 NULL 都不序列化
> 
> JsonInclude.Include.NON_NULL      属性为NULL  不序列化
> ```
>
> 

使用之前

![img](https://images2017.cnblogs.com/blog/727732/201710/727732-20171011233115043-1083920724.png)

使用之后

![img](https://images2017.cnblogs.com/blog/727732/201710/727732-20171011232957965-1037343399.png)

方法二、 如果不想每次都这样添加，可以在application.yml配置全局定义， 这种默认都生效

```yaml
spring:
 
   jackson:
 
        default-property-inclusion: non_null
```

 

方法三、通过ObjectMapper 对象进行设置，下面是测试用例

```
@Test
public  void  test() throws JsonProcessingException {
    ResultVo resultVo = new ResultVo();
    resultVo.setCode(0);
    resultVo.setMsg("成功");
 
    ObjectMapper mapper = new ObjectMapper();
    mapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);//默认
    String json = mapper.writeValueAsString(resultVo);
    System.out.println(json);
 
    mapper = new ObjectMapper();
    mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL); //属性为NULL不序列化
    json = mapper.writeValueAsString(resultVo);
    System.out.println(json);
 
    Map map = new HashMap();
    map.put("code",0);
    map.put("msg","成功");
    map.put("data",null);
    json = mapper.writeValueAsString(map);
    System.out.println(json);
}
```

打印如下：

```
{"code":0,"msg":"成功","data":null}
 {"code":0,"msg":"成功"}
 {"msg":"成功","code":0,"data":null}
```

注意：ObjectMapper 只对VO起作用；对Map List不起作用

 

1、如果必定返回的字段，可以在实体类一开始就给默认值（如字符串 ”” ； list [] ），来避免null

2、jackson实体转json时，某个属性不参加序列化时 使用@JsonIgnore 放在该属性上