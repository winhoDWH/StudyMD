### 配置集 `Configuration Set`
通常一个配置文件就是一个配置集

### 配置集ID `Date ID` 
给一个配置集进行唯一标记保证全局唯一性；（即对一个配置文件进行唯一标识）

### 配置组 `Group`
用于区分`Data ID`相同的配置集，默认使用`DEFAULT_GROUP`；

### 服务Service

### 服务注册中心

### 服务发现

### 元信息

### 应用

### 服务实例

### 虚拟集群

### 集群

### 客户端


### 获取配置
#### `@NacosValue标签`
1. `@NacosValue(value = "${配置名:默认值}", autoRefreshed = false)`；默认值可以省略，但如果没有设置默认值，且nacos中没有对应配置，则启动项目时会报错；
2. `autoRefreshed`属性为true时，可以在不重启项目的情况下，修改了nacos的配置，实现动态配置；但这个

#### `springboot`中使用nacos配置信息
##### 第一种方式
1. 官网的方式：
   * 在启动类上使用`@NacosPropertySource`标签声明加载nacos配置；
   * 然后使用`@NacosValue`或者`@Value`注入nacos中对应`Data ID`的配置；
   * 该方式的问题：不能使用多个配置集`dataID`；nacos相关配置：`name/groupId/dataId/autoRefreshed/type`（点开`@NacosPropertySource`注解可以看到具体有哪些配置项）都是要在该注解中配置的，即代码中写入；但其他如：namespace这些可以在配置文件yml中使用nacos.config前缀配置；（其实就是`@NacosPropertySource`注解中`properties`属性）；

2. 直接使用`NacosConfigProperties`进行自动配置
   * 在yml配置文件中，将`nacos.config.bootstrap.enable`设置为true；则可以触发spring的自动配置nacos；
   * springboot启动日志：
     * 没有设置的（即默认为false）：
        ```
        2021-05-28 17:01:46.700  INFO 14180 --- [           main] c.a.b.n.c.u.NacosConfigPropertiesUtils   : nacosConfigProperties : com.alibaba.boot.nacos.config.properties.NacosConfigProperties@6e0f5f7f
        2021-05-28 17:01:46.702  INFO 14180 --- [           main] NacosConfigApplicationContextInitializer : [Nacos Config Boot] : The preload configuration is not enabled
        ```
      * 设置了true的：
        ```
        2021-05-28 17:12:17.705  INFO 19516 --- [           main] c.a.b.n.c.u.NacosConfigPropertiesUtils   : nacosConfigProperties : com.alibaba.boot.nacos.config.properties.NacosConfigProperties@71a8adcf
2021-05-28 17:12:17.774  INFO 19516 --- [           main] c.a.b.n.config.util.NacosConfigUtils     : load config from nacos, data-id is : example1, group is : DEFAULT_GROUP
        ```
  * 然后使用`@NacosValue`或者`@Value`注入nacos中对应`Data ID`的配置；

#### 两种方式的共同点
1. 想要动态的配置，必须使用`@NacosValue`注解注入配置值，并且`autoRefreshed=true`;`@NacosPropertySource`与yml配置中`nacos.config.auto-refresh`设置为true是一个全局的开关标识，配和`@NacosValue`的`autoRefreshed=true`一起才能实现动态配置；
2. `@Value`注入配置值，不会有动态配置的效果；

#### 优先级
两种方法一起使用的时候，第二种方法优先级最高；如果两种方法的nacos配置文件(`dataID`)中有同名的配置项，则以第二种方式（`NacosConfigProperties自动配置`）为主，就算是动态改变第一种方法中的nacos配置文件中的配置项，也不会对读取有改变；只有第二种方法的配置文件中没有的配置项而第一种方法中有，才会取第一种方法的配置项的值；