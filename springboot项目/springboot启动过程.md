### META-INF/spring.factories文件作用
1. 背景：springboot在启动的时候会通过`@SpringBootApplication注解`中的`@ComponentScan注解`去扫描当前目录下所有`class`，然后实例化，但是**根目录以外的bean应该如何加载呢？比如maven依赖中的，这时候就需要`META-INF/spring.factories`**;
2. 所以该文件相当于一个描述文件，描述需要的

参考：
1. https://blog.csdn.net/qq_35549286/article/details/109047777 META-INF/spring.factories文件的作用是什么
2. https://www.jianshu.com/p/8dff56465dbf SpringBoot 启动过程源码分析
3. 