## 启动

1. 通过**`@EnableAspectJAutoProxy`** 注解开开启AOP；
1. 查看**`@EnableAspectJAutoProxy`** 注解发现这是通过@import注解注入到spring的IOC容器里面的；其中使用了**`AspectJAutoProxyRegistrar`** 作为import注解的注入手段；
1. 通过@import注解在IOC容器中生成**`AnnotationAwareAspectJAutoProxyCreator `**实例



## 动态生成切面实例

1. 通过**`AnnotationAwareAspectJAutoProxyCreator `**类重写了 **`postProcessAfterInitialization方法和postProcessBeforeInstantiation方法`** 

2. 调用 **`AbstractAutoProxyCreator#postProcessBeforeInstantiation`** 方法查找所有的Advisor和切面，将切面构建成Advisor

3. 调用 **`AbstractAutoProxyCreator#postProcessAfterInitialization`** 方法，从缓存取出所有的将所有的增强器，创建代理工厂，并织入增强器，创建代理对象

## 参考

1. Spring AOP 源码解析：https://juejin.cn/post/7037839773267918879