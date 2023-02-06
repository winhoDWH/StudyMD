## 介绍

官方推荐对java语言支持的工具，还有Redisson。

## 连接redis单机版本

### 一、基本操作的实例-Jedis对象

- 构造方法：new jedis("ip", "port");
- 其余操作方法名的与redis类型的操作指令基本一致；

```java
//创建Jedis连接对象
Jedis jedis = new Jedis("localhost", 6379);
//添加string类型
jedis.set("person", "张三");
```

## 二、使用Jedis连接池

- 配置类：JedisPoolConfig
  - **setMaxTotal()**：设置连接池最大连接数；
  - **setMaxWaitMillis()**：设置连接对象Jedis最长等待时间；

- 连接池类：JedisPool
  - 构造函数：JedisPool(配置对象,服务器名,端口号)
  - Jedis getResource()：获取一个Jedis对象；
  - void close()：关闭连接池。
- 示例：

```java
JedisPoolConfig config = new JedisPoolConfig();
//最大链接数
config.setMaxTotal(10);
//最大连接时间
config.setMaxWait(Duration.ofMillis(10000));	//4.0.0-SNAPSHOT版本
//config.setMaxWaitMillis(1000);                //老版本可用
JedisPool pool = new JedisPool(config, HOST, port);
//获取一个操作对象
Jedis pool.getResource();
//结束线程池
pool.close();
```



## 连接redis多集群

### 操作对象-JedisCluster实例

- 待补充。

## 参考

https://blog.csdn.net/weixin_45481821/article/details/125009912?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166175821516781683911721%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=166175821516781683911721&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~pc_rank_34-10-125009912-null-null.142^v42^pc_rank_34,185^v2^control&utm_term=jedis&spm=1018.2226.3001.4187