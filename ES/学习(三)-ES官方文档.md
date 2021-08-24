### ES-Search API的理解与说明
官方文档连接：https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-api-query-params-q

#### 标题讲解
##### Request
使用请求Request去查询ES数据的方法有四种：
```
Request
    GET /<target>/_search

    GET /_search

    POST /<target>/_search

    POST /_search
```
##### Description
说明允许使用请求去查询ES，如果想对数据进行过滤，则使用**下文中Query parameter的q属性**设置查询条件或者使用**下文request body**设置查询条件；

##### Path parameters
说明的是<target>，即访问地址，其实就是(索引名indexName/类型名typeName)组成；

##### Query parameters
说明的是get请求url后添加的参数，即GET /<target>/_search?后面拼接什么参数，有什么用；

##### Request body
1. "from": integer；从第几个开始查询，**默认是0**，即**ES第一个文档数据的索引编号为0**；
2. "size": integer; 返回多少条数据；默认使用from和size两个属性时，不能查询超过10000条文档，这个限制一般使用index.max_result_window这个属性在索引做设置；
3. "query":{....},这里是Query DSL，具体看下文；

#####
