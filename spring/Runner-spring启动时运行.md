## 实现Spring启动后执行某些初始化操作--CommandLineRunner以及ApplicationRunner

​	作用：如果想在Spring工程在启动之后自动运行某些操作，则可使用以上两种Runner实现。只需要继承之后实现对应方法即可

## 一、 CommandLineRunner

1. 继承后将初始化操作写入run方法中即可。

```java
public interface CommandLineRunner {
    //args就是main函数（工程启动）的参数
    void run(String... args) throws Exception;
}

//自定义runner类
//加上bean注解，使其能被扫描发现并注入到bean容器中
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        //init操作
    }
}
```

## 二、ApplicationRunner

与CommandLineRunner同理。

```java
@FunctionalInterface
public interface ApplicationRunner {
    //这里的args同样也是工程启动时输入参数，不过使用ApplicationArguments封装
    void run(ApplicationArguments args) throws Exception;
}
```

## 三、调整多个runner之间的运行顺序

​	使用@Order或者实现`org.springframework.core.Ordered`接口

## 参考

1. https://juejin.cn/post/6847902216037072904