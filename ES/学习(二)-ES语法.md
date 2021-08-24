### ES语法
ES由于支持Restfulweb风格的url访问，实际上可以理解成使用web请求调用其中服务；所以都是使用http的方式进行数据操作；

### 查看ES版本
使用GET请求，请求对应ES所在机器ip的9200端口即可，即：curl -XGET localhost:9200，看到
```
{
    "name": "server2093",
    "cluster_name": "bigdata",
    "version": {
        "number": "2.1.2",
        "build_hash": "NA",
        "build_timestamp": "NA",
        "build_snapshot": true,
        "lucene_version": "5.3.1"
    },
    "tagline": "You Know, for Search"
}
```
看到version中标识的number字段即为ES的版本；

#### 新增数据
1. 使用PUT请求：
    ```
    PUT /megacorp/employee/1
    {
        "first_name" : "John",
        "last_name" :  "Smith",
        "age" :        25,
        "about" :      "I love to go rock climbing",
        "interests": [ "sports", "music" ]
    }
    ```
    * 上面是新增ES数据，后面的/1是数据的ID，**使用PUT请求就一定要传该参数**
2. 使用POST请求：
    ```
    POST /megacorp/employee
    {
        "first_name" : "John",
        "last_name" :  "Smith",
        "age" :        25,
        "about" :      "I love to go rock climbing",
        "interests": [ "sports", "music" ]
    }
    ```
    * 使用POST请求就不需要加上ID，ES会自动生成指定ID；
3. 如果想对该数据进行更新操作，则重新请求该URL（**使用PUT还是POST请求？**），**关键是ID一定要指定**，然后修改json内容；

#### 检索数据
1. 使用GET请求，如下例子中，/megacorp/employee是ES的某个索引中的某个type的访问url，1是ID，这句是查询对应ID的文档信息；
    ```
        请求：
        GET /megacorp/employee/1
        响应：
        {
            "_index": "megacorp",    //文档所在索引
            "_type": "employee",     //文档所在type
            "_id": "1",              //文档的ID号
            "_version": 1,           //版本号
            "found": true,           //是否查找成功
            "_source": {
                "first_name": "John",
                "last_name": "Smith",
                "age": 25,
                "about": "I love to go rock climbing",
                "interests": [
                    "sports",
                    "music"
                ]
            }
        }
    ```
    * 看上面响应，这是显示了查询一个文档信息时返回的内容，**其中"_source"是我们的存储的Json信息**，而其他键值对的值为文档的一些属性信息；
2. 简单检索：
    ```
    请求：
    GET /megacorp/employee/_search
    响应：
    {
        "took": 1,                             //耗时
        "timed_out": false,                    //是否超时
        "_shards": {
            "total": 2,
            "successful": 2,
            "failed": 0
        },
        "hits": {
                "total": 14,                    //总文档数
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "megacorp",    //文档所在索引
                        "_type": "employee",     //文档所在type
                        "_id": "1",              //文档的ID号
                        "_version": 1,           //版本号
                        "found": true,           //是否查找成功
                        "_source": {
                            "first_name": "John",
                            "last_name": "Smith",
                            "age": 25,
                            "about": "I love to go rock climbing",
                            "interests": [
                                "sports",
                                "music"
                            ]
                        }
                    },......
                ]
        }
    }
    ```
    * 使用_search进行检索，检索所有文档，默认**显示10条**，可以通过size参数改变该设置（使用?后面加参数的形式），也可以通过设置from参数指定位移（默认从第0条开始）。
    * 返回字段中，took表示操作耗时（**单位为毫秒**），time_out表示是否超时，hits字段表示命中记录
3. 简单条件检索：
    ```
    //检索last_name=Smith的文档
    GET /megacorp/employee/_search?q=last_name:Smith
    ```

#### 复杂条件检索
例子：
```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```
1. 使用json串并在json串中加入语法，便可以查询更复杂的条件；

##### query context
1. 全文本查询：分类-模糊匹配（关键词match）、习语匹配（match_phrase）、多字段匹配（multi_match）、语法查询
2. 字段级别查询：

###### 全文本查询-关键词match
1. 该关键词针对**文本类型数据**进行查询，称为**模糊查询**，但是和SQL中字符串带*的模糊查询不同，这里的模糊查询是：**ES会对输入的匹配文本进行分词（分词机制暂时不知），然后根据分词之后的单词进行检索，即当检索字段中有分词后的某个单词时，就被匹配**；
2. 例子： 
```
    {
        "query":{
            "match":{
                "content":"中国广州"
            }
        }
    }
```
首先会对检索条件中的'中国广州'进行分词，分成'中国'和'广州',然后匹配Content字段中包含这两个单词之一或者两者都有的文档；如content="中国真美"，content="广州真大"，content="中国很美，广州很大"这三种记录都会被选出来；

###### 全文本查询-match_phrase 习语查询关键词？

###### 全文本查询-multi_match 多字段查询
```
    {
        "query":{
            "multi_match":{
                "query":"万里",
                "fields":["author","content"]
            }
        }
    }
```
上面例子查询"author"和"content"两个字段中包含"万里"的文档。**所以该关键词可以看成有两个属性**
1. **query属性**：该属性说明要匹配的字段文本值；
2. **fields属性**：该属性说明匹配字段名；

###### 全文本查询-query_string 语法查询
1. 根据一定语法规则进行查询。支持通配符、防伪查询、布尔查询、正则表达式

###### 字段查询-查询特定项
```
{
    "query":{
        "term":{
            "word_count":1000
        }
    }
}
```
* 使用**关键字term**进行查询；上面例子是精确查询word_count字段为1000的文档；

###### 字段查询-范围查询
```
{
    "query":{
        "range":{
            "word_count":{
                "gte":1000,
                "lte":2000
            }
        }
    }
}
```
1. 使用**关键字range**进行查询；上面例子是查询word_count字段的值在1000到2000的文档
2. 查询日期类型字段同样**可用range关键字进行查询**，其中可用now代表现在时间；
2. 该范围查询，不止适用于数值类型的字段，字符类型字段也适用（？原理是什么）；

#### 查询体
1. 由一个json组成，json中一级目录：query字段代表检索查询条件，在query字段中使用Query DSL语法
```
{
    "query":{
        .......
    },
    "别的字段"：...
}
```
2. 其他一级目录字段：
    * sort代表排序
        ```
        "sort": [
            {
                "age": {
                    "order": "desc"
                }
            }
        ]
        ```
    * 分页查询：from和size两个字段
        ```
            "from": 0,
            "size": 1
        ```

##### query字段简易概况-一级目录
下面列出query中会出现的字段以及其作用；
###### match字段
匹配文本，根据es中分词器对检索条件中需匹配的文本进行分词，分词后再进行匹配
```
//查询name字段中，"admin"该文本被分词后，能被匹配到的文档记录
{
   "query":{
        "match":{
            "name": "admin"   //其实就是查询条件
        }
    } 
}
```

###### term字段
精确匹配，检索结果一定要与该条件中的值完全一致 
```
//查询name字段完全匹配admin（即name=admin）的文档记录
{
    "query":{
        "term":{
            "name": "admin"   //其实就是查询条件
        }
    }
}
```

###### 数值区间
```
gt 大于
gte 大于等于
lt 小于
lte 小于等于！
例子：
{
    "query":{
        "range":{
            "age":{
                "gt": 20,
                "lt": 40
            }
        }
    }
}
```

###### 布尔值查询
1. must字段（and）,其值是一个数组形式，数组里的元素其实就是一个个检索条件（即match和term的任意个数组合）,如果只有一个检索条件则使用{}就行
```
//查询姓名（name）中匹配admin,并且性别sex=1的记录
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "name":"admin"
                    }
                },{
                    "term":{
                        "sex":1
                    }
                }
            ]
        }
    }
}
```
2. should字段(or)，其值也是一个数组形式;
```
//查询姓名（name）中匹配admin或者性别sex=1的记录
{
    "query":{
        "should":[
            {
                "match":{
                    "name":"admin"
                }
            },{
                "term":{
                    "sex":1
                }
            }
        ]
    }
}
```
3. must_not （not），其值也是一个数组；（待测其数组中有多个检索条件时，是怎么判断的）

###### filter过滤



### 参考
1. https://blog.csdn.net/u011499747/article/details/78877922?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160439541219724822511963%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=160439541219724822511963&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v28-2-78877922.pc_search_result_cache&utm_term=es%E8%AF%AD%E6%B3%95&spm=1018.2118.3001.4449 Elasticsearch 基本语法汇总
2. https://blog.csdn.net/qwqw3333333/article/details/78261469?utm_medium=distribute.pc_relevant.none-task-blog-utm_term-2&spm=1001.2101.3001.4242 （九）ElasticSearch高级查询语法
3. https://blog.csdn.net/qwqw3333333/article/details/78255996 （八）ElasticSearch常用查询语法
4. https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html 官方文档
5. https://blog.csdn.net/lianxiaobao/article/details/79115263?utm_source=blogxgwz7&utm_medium=distribute.pc_relevant.none-task-blog-utm_term-7&spm=1001.2101.3001.4242 
6. https://blog.csdn.net/qq_40649503/article/details/109572150?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160508400519724835808708%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=160508400519724835808708&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v28-14-109572150.pc_search_result_cache&utm_term=springboot%E4%B8%AD%E6%9C%89%E5%A4%9A%E4%B8%AA%E6%95%B0%E6%8D%AE%E6%BA%90&spm=1018.2118.3001.4449 狂神聊 ElasticSearch