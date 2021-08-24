### 阿里的fastjson的使用
1. Fastjson 源码地址：https://github.com/alibaba/fastjson
    Fastjson 中文 Wiki：https://github.com/alibaba/fastjson/wiki/Quick-Start-CN

2. 依赖：
    ```
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>x.x.x</version>
    </dependency>
    ```
3. json是一种文本，具体程序表现是一种特殊格式的字符串，是**名称/值对**的形式。
    >{"name":"dwh"},**名称与值之间是冒号！**
    >而map的对应字符串格式是：{name=dwh};**这种用等号的是键值对形式**；

4. java中JSON分为，JSON类、JSONObject类、JSONArray类
    * JSONObject类是json的对象类；JSONArray是JSONObject的数组类；JSON类是**主要用于提供json和java对象之间的转换方法的类**
    * 注意要分清：**json字符串和json对象（JSONObject对象）**；
    * 关于json字符串的操作：
        1. json字符串转java对象：实际上就是一个反序列化的过程，使用**JSON.parseObject()（默认转成JSONObject对象，添加参数.class可以转成特定的java对象）或者JSON.parse()方法**;可以通过@JSONField标签控制转换的对象中个成员变量的值等；
        2. java对象转json字符串：是一个序列化的过程，**使用JSON.toJSONString()方法**；
    * 关于JSONObject对象的操作：
        1. JSONObject继承了JSON类，实现了Map<String, object>（可以强转），所以可以使用get，put方法进行json的拼装；
        2. JSONObject转json字符串：JSON.toJSONString()方法；
        3. JSONObject获取JSONArray与其中某个值：getXXX();
    * JSONArray类实现了List<Object>
5. 由于有泛型存在，所以json对象转成带泛型的集合还是最好使用遍历的方式；

6. @JSONField标签
    * 该标签加在javabean中的成员变量上；可加可不加，对象都可以被序列化成json字符串
    * 加了标签之后可以修改序列化后的命名等
    * name属性：起别名；   serialize属性：false指定字段不序列化；
    * ordinal属性：规定字段序列化后的顺序；（默认的序列化顺序是根据name的字母顺序）
    * format属性：用于格式化date属性
    * 如果属性是私有的，就必须有set方法，否则无法反序列化
    * 必须要有无参数的构造器，不然反序列化会报错：JSONException: default constructor not found.

7. json字符串转换成JSONObject时的情况
    * 第一种：对于null、空字符串、转义字符（如换行符）这样的字符串，使用JSON.parseObject进行转换**不会报错**，但是**转换后的对象为NULL**;
    * 第二种："{}"与"[]"这两种分别转换成json对象和数组时，使用size()==0进行判断。（**相当于容器判断，因为JSONObject和JSONArray本质上就一个Map和List**）；
    * 第三种："a"等不符合JSON书写（**键值对通过:冒号连接**）的字符串，则在转换时会报错；