### Mapping Types
每一个mapping类型都有meta-fields与（fields or prperties)；在一个索引中，不同的mapping里同名的字段必须有相同的映射；

#### 字段类型
1. 简单类型：String,data,long.double,boolean和ip
2. 支持json类型的数据类型，即object这种

不能更新已有的mappings

### 数据类型
1. 字符类型：string
2. 数值类型：long,integer,short,byte,double,float
3. 日期类型：date
4. 布尔类型：boolean
5. 二进制类型：binary
6. 数组类型，不指定数组内数据类型，但同一数组内数据类型要一致（不同文档，同一字段如果一个是字符串数组一个是数值数组呢），空数组[]会被视为缺少字段？

### 创建Mapping
1. 使用**PUT请求**创建Mapping，分三种情况
   * 一是没创建了索引index，使用`mappings`：
        ```
        创建一个索引twitter，并定义索引的mapping映射tweet中有一个叫message的字符串类型字段
        PUT host:port/twitter
        {
            "mappings": {
                "tweet": {                   --->[mapping名称]
                    "properties": {
                        "message": {         --->[字段名]
                            "type": "string" --->[属性类型]
                        }
                    }
                }
            }
        }
        ```
    * 二是创建了索引，使用`_mapping`：
        ```
        为twitter索引新增一个新的叫user的mapping映射，其中有名为name且类型为字符串的字段
        PUT twitter/_mapping/user 
        {
        "properties": {
            "name": {
            "type": "string"
            }
        }
        }
        ```
     * 三是为已有Mapping映射新增字段：
        ```
        新增了一个叫user_name的字段
        PUT twitter/_mapping/tweet 
        {
        "properties": {
            "user_name": {
            "type": "string"
            }
        }
        }
        ```
2. 同一个索引下，不同的mapping之间如果有相同的字段，由于要保持其字段属性一致（除了某些属性外），不能直接更新，可以在更新请求后添加`?update_all_types`，使两个mapping相同的字段属性一起更新
mapping parameter

3. mapping的新增的语法结构：
   ```
   {
       "mappings":{
           "索引名":{
               "meta-fields":{ }
               "properties":{
                   "字段名":{
                       "type":"类型"
                       "mapping parameters":....
                   }
               }
           }
       }
   }
   ```