## @Conditional注解

1. Spring4新提供的注解，作用：**按一定条件判断是否往容器中注入bean**。
2. 有一个参数value数组，需要传入继承了Condition接口的数组；

### conditional接口

需要实现matches接口，返回true注入bean，否则不注入

- 一个方法只能注入一个bean实例，所以@Conditional标注在方法上只能控制一个bean实例是否注入。

- 一个类中可以注入很多实例，@Conditional标注在类上就决定了一批bean是否注入。
- 多个判断class的时候，暂时看示例是逻辑与的关系。

## 参考

##### Spring @Conditional注解 详细讲解及示例：https://blog.csdn.net/xcy1193068639/article/details/81491071

