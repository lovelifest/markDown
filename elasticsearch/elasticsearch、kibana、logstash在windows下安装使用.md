## elasticsearch\kibana\logstash在windows下安装使用

### elasticsearch

### elasticsearch-head

1. ​	安装node.js （官网下载）

2. 检查node.js版本

   ```
   node --version
   ```

3. 进入node.js目录安装grunt

   ```
   ##安装grunt
   npm install -g grunt-cli
   ##查看版本
   grunt -version
   ```

4. 启动elasticsearch-head

5. 会存在跨域问题，所以要在elasticsearch中配置跨域在elasticsearch.yml中

   ```
   http.cors.enabled: true 
   http.cors.allow-origin: "*"
   node.master: true
   node.data: true
   
   然后去掉
   network.host: 192.168.0.1
   的注释并改为
   network.host: 0.0.0.0
   去掉
   cluster.name
   node.name
   http.port
   的注释（也就是去掉#）
   ```

6. 在下载好的head文件夹内找到Gruntfile.js，在对应的位置加上hostname: ‘\*’

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190216145811924.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODc1NjY3,size_16,color_FFFFFF,t_70)

7. 启动

   ```
   grunt server
   ##或者
   npm run start
   ```

8. 访问

   ```
   http://localhost:9100/
   ```
### kibana

### logstash





