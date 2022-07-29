spring优点：

1. 开源、免费的框架
2. 轻量级、非入侵的框架

3. 控制反转IOC、面向切面编程AOP
4. 支持事务处理

## IOC

1. 通过Set方法，将程序从主动创建某个实现类，变成被动接受

## bean别名

特殊情况：

​	当`beanA`的ID与`beanB`的别名同名时，会优先根据别名找到`beanB`的ID，从而找到`beanB`的实例；原因看获取`beanId`的源码：

```java
/**
版权声明：本文为CSDN博主「康桑米拉达」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq896413023/article/details/121947395
*/
类名：SimpleAliasRegistry
/**
 * Determine the raw name, resolving aliases to canonical names.
 * @param name the user-specified name
 * @return the transformed name
 */
public String canonicalName(String name) {
	String canonicalName = name;
	// Handle aliasing...
	String resolvedName;
	do {
		resolvedName = this.aliasMap.get(canonicalName);
		if (resolvedName != null) {
			canonicalName = resolvedName;
		}
	}
	while (resolvedName != null);
	return canonicalName;
}
```

​	由源码可知`beanId`都是优先走根据别名获取对应ID的。

实验：

`bean.xml`文件中

```xml
	<bean id="bean1" class="dwh.entity.TestBean">
        <constructor-arg index="0" value="s1"/>
    </bean>

    <bean id="bean2" class="dwh.entity.TestBean">
        <constructor-arg name="str" value="s2"/>
    </bean>
	<alias name="bean1" alias="bean2"/>
```

测试类：

```java
public class BeanTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
		
       	//看获取到的实例是bean1还是bean2
        TestBean test = (TestBean) context.getBean("bean2");
        System.out.println(test.toString());
    }
}
```

输出：

```
#输出参数为S1说明为bean1的实例
TestBean{str='s1'}
```

