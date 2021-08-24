### 连接ES
1. 依赖：
   ```
   <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <!--本系列使用2.1.2版本-->
            <version>${es.version}</version>
    </dependency>
   ```
2. 使用TransportClient创建客户端client，有两种方式
   * 集群名为默认的elasticsearch的时候
    ```
    Client client = TransportClient.builder().build()
                .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("ip"), 9300));
    ```
   * 集群名自定义的时候
    ```
    //自定义了一个集群名为bigdata,需要创建一个settings对象
    Settings settings = Settings.settingsBuilder().put("cluster.name", "bigdata").build();
    
    //将settings对象通过.settings(settings)方法设置到client中
    client = TransportClient.builder().settings(settings).build().addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("ip"), 9300));
    ```
    client可以使用多次`.addTransportAddress(ip对象, 端口)`方法来设置部署成集群模式且在多台机器上的es；
