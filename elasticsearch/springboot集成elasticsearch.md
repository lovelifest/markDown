###### pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.funtl.st</groupId>
	<artifactId>java-api</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>java-api</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<!--elasticsearch版本-->
		<elasticsearch.version>7.7.0</elasticsearch.version>
		<fastjson.version>1.2.72</fastjson.version>
	</properties>

	<dependencies>
		<!--elasticsearch依赖-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>${fastjson.version}</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



```
package com.funtl.st.config;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author songtao
 * @create 2020-07-2020/7/4-13:21
 */
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
package com.funtl.st;

import com.alibaba.fastjson.JSON;
import com.funtl.st.pojo.User;
import net.minidev.json.JSONArray;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.TermQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.FetchSourceContext;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.ArrayList;

@SpringBootTest
class JavaApiApplicationTests {

    @Autowired
    @Qualifier("restHighLevelClient")
    private RestHighLevelClient client;

    @Test
    void contextLoads() {
    }

	/**
	 * 创建index
	 * @throws IOException
	 */
	@Test
    public void createIndex() throws IOException {
		CreateIndexRequest createIndexRequest = new CreateIndexRequest("songtao");
		CreateIndexResponse createIndexResponse = client.indices().create(createIndexRequest, RequestOptions.DEFAULT);
		boolean acknowledged = createIndexResponse.isAcknowledged();
		System.out.println(acknowledged);
    }

	/**
	 * 索引是否存在
	 * @throws IOException
	 */
	@Test
	public void existIndxe() throws IOException {
		GetIndexRequest getIndexRequest = new GetIndexRequest("songtao");
		boolean exists = client.indices().exists(getIndexRequest, RequestOptions.DEFAULT);
		System.out.println(exists);
	}

	/**
	 * 删除索引
	 * @throws IOException
	 */
	@Test
	public void delIndex() throws IOException {
		DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("songtao");
		AcknowledgedResponse acknowledgedResponse = client.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);
		System.out.println(acknowledgedResponse.isAcknowledged());
	}

	/**
	 * 创建doc
	 */
	@Test
	public void putDoc() throws IOException {
		User user = new User("songtao",20);
		IndexRequest indexRequest = new IndexRequest("songtao");
		indexRequest.id("1");
		indexRequest.timeout(TimeValue.timeValueSeconds(1));
		indexRequest.timeout("1s");
		indexRequest.source(JSON.toJSONString(user), XContentType.JSON);
		IndexResponse response = client.index(indexRequest, RequestOptions.DEFAULT);
		System.out.println(response.status());
	}

	/**
	 * 判断文档是否存在
	 */
	@Test
	public void  existDoc() throws IOException {
		GetRequest getRequest = new GetRequest("songtao", "1");
		//不获取返回的_source的上下文了
		getRequest.fetchSourceContext(new FetchSourceContext(false));
		getRequest.storedFields("_none_");
		boolean exists = client.exists(getRequest, RequestOptions.DEFAULT);
		System.out.println(exists);
	}

	/**
	 * 获取文档
	 */
	@Test
	public void getDoc () throws IOException {
		GetRequest getRequest = new GetRequest("songtao", "1");
		GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
		System.out.println(getResponse.getSourceAsString());
		System.out.println(getResponse);
	}

	/**
	 * 更新文档
	 */
	@Test
	public void updatDoc() throws IOException {
		UpdateRequest updateRequest = new UpdateRequest("songtao", "1");
		updateRequest.timeout("1s");
		updateRequest.timeout(TimeValue.timeValueSeconds(1));
		User songtao1 = new User("songtao1", 3);
		updateRequest.doc(JSON.toJSONString(songtao1), XContentType.JSON);
		UpdateResponse updateResponse = client.update(updateRequest, RequestOptions.DEFAULT);
		System.out.println(updateResponse.status());
	}

	/**
	 * 删除文档
	 */
	@Test
	public void delDoc() throws IOException {
		DeleteRequest deleteRequest = new DeleteRequest("songtao", "1");
		deleteRequest.timeout("1s");
		deleteRequest.timeout(TimeValue.timeValueSeconds(1));
		DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
		System.out.println(deleteResponse.status());
	}

	/**
	 * 批量插入doc
	 */
	@Test
	public void bulkDocs() throws IOException {
		BulkRequest bulkRequest = new BulkRequest();
		bulkRequest.timeout("1s");
		ArrayList<User> users = new ArrayList<>();
		users.add(new User("st1", 1));
		users.add(new User("st2", 2));
		users.add(new User("st3", 3));
		users.add(new User("st4", 4));
		users.add(new User("zhangsan1", 1));
		users.add(new User("zhangsan2", 2));
		users.add(new User("zhangsan3", 3));
		users.add(new User("zhangsan4", 4));
		for (int i = 0; i < users.size(); i++) {
			bulkRequest.add(
					new IndexRequest("songtao")
							.id("" + (i + 1))
							.source(JSON.toJSONString(users.get(i)), XContentType.JSON)
			);
		}
		BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);
		boolean b = bulkResponse.hasFailures();
		System.out.println(b);
	}

	/**
	 *  查询
	 *  SearchRequest 搜索请求
	 *  SearchSourceBuilder 条件构造
	 *  HighlightBuilder 构建高亮
	 *  TermQueryBuilder 精确查询
	 *  MatchAllQueryBuilder
	 *  xxx QueryBuilder 对应我们刚才看到的命令！
	 */
	@Test
	public void searchDoc() throws IOException {
		SearchRequest searchRequest = new SearchRequest("songtao");
		SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
		searchSourceBuilder.highlighter();
		// 查询条件，我们可以使用 QueryBuilders 工具来实现
		// QueryBuilders.termQuery 精确
		// QueryBuilders.matchAllQuery() 匹配所有
		TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "st3");
		searchSourceBuilder.query(termQueryBuilder);
		searchSourceBuilder.timeout(TimeValue.timeValueSeconds(1));
		searchRequest.source(searchSourceBuilder);
		SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
		String s = JSON.toJSONString(searchResponse.getHits());
		System.out.println(s);
		System.out.println("====================================");
		for (SearchHit documentFields : searchResponse.getHits().getHits()) {
			System.out.println(documentFields.getSourceAsMap());
		}
	}
}

```

