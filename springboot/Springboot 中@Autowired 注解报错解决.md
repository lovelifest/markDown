### Springboot 中@Autowired 注解报错解决

1. ###### 保证配置类和Autowired的名字一致

   

   ```java
   //配置类
   @Configuration
   public class ElasticsearchConfig {
       @Bean
       public RestHighLevelClient restHighLevelClient(){
           RestHighLevelClient client = new RestHighLevelClient(
                   RestClient.builder(
                           new HttpHost("192.168.95.10", 9200, "http")));
           return client;
       }
   }
   ```

   ```java
   @Autowired
   private RestHighLevelClient restHighLevelClient;//和配置类的名称一致
   ```

2. 增加注解

   ```java
   @Autowired
   @Qualifier("restHighLevelClient")//和配置类的名称保持一致
   private RestHighLevelClient client;//可以随意自定义
   ```

   

