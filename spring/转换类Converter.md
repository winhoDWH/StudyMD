## spring框架中的数据转换

### Converter接口
```
public interface Converter<S, T> {
   
    T convert(S source);
 
}
```
1. 主要方法是使用convert方法将第一个类型S转换为第二种类型T，当需要建立自己的转换类时，可以继承该类并重写convert方法；

### 参考
1. https://blog.csdn.net/iteye_13139/article/details/82507079?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control