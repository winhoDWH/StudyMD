## import注解

1. 作用：快速的为spring容器注册组件

2. 一共有三种方式：

   - 导入`@Configuration`注解的配置类使用`@Import`（要导入到容器中的组件）：容器中就会自动注册这个组件，ID默认为全类名

   - 导入`ImportSelector`的实现类：通过实现`ImportSelector`类，实现`selectImports`方法，返回需要导入组件的全类名数组

   - 导入`ImportBeanDefinitionRegistrar`的实现类：通过实现`ImportBeanDefinitionRegistrar`类，实现`registerBeanDefinitions`方法手动注册Bean到容器中

### 第一种 @Configuration注解

1. 这种就是简单的在`@Import`注解的属性value中加入需要注入容器的类class对象即可。
2. 使用实例：

```java
// 待注入的User
public class User {
}

// 配置类
@Configuration
@Import(User.class)     //使用@Import导入组件，ID默认是组件的全类名
public class AppConfig {
}

// 启动类，通过打印容器中的Bean来判断是否注入
@Test
public void TestMain(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    String[] beanNames = applicationContext.getBeanDefinitionNames();
    for (String beanName : beanNames) {
        System.out.println(beanName);
    }
}
```



### 第二种 使用`ImportSelector`实现类

1. 自定义一个`ImportSelector`实现类，实现其中`selectImports`方法

```java
 /**
     * @description 获取要导入到容器的组件全类名
     * @author ONESTAR
     * @date 2021/2/25 15:49
     * @param annotationMetadata：当前标注@Import注解类的所有注解信息
     * @throws
     * @return java.lang.String[]
     */
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{"bean.Person"};
    }
```

3. 使用：

   ```java
   //自定义ImportSelector实现类
   public class MyImportSelector implements ImportSelector {
       public String[] selectImports(AnnotationMetadata annotationMetadata) {
           //待注入类的类路径
           return new String[]{"bean.Person"};
       }
   }
   
   // 待注入的Person
   public class Person {
   }
   
   // 配置类
   @Import({MyImportSelector.class})     //使用@Import导入组件，ID默认是组件的全类名
   public class AppConfig {
   }
   
   //spring启动类
   @Test
   public void TestMain(){
       AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
       String[] beanNames = applicationContext.getBeanDefinitionNames();
       for (String beanName : beanNames) {
           System.out.println(beanName);
       }
   }
   ```

   

4. 理解：该方式简单来说就是将`selectImports`方法返回的类注入到spring容器中，可以通过`AnnotationMetadata`获取当前注解的信息从而判断生成注入什么类实例。

5. 默认生成的实例ID也是类全名（带路径）。**这里`@Import`的value属性直接使用selectImports实现类的class对象即可**。

### 第三种 导入`ImportBeanDefinitionRegistrar`的实现类

1. `ImportBeanDefinitionRegistrar`类代码：

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	* annotationMetadata:同上面selectImports一样是引入该类的注解信息
	* beanDefinitionRegistry：bean注册器，通过该注册器注册bean
	*/
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {

        // 指定Bean的名称
        RootBeanDefinition beanDefinition = new RootBeanDefinition(Animal.class);
        //通过beanDefinitionRegistry实例将类实例注册到springIOC容器中
        beanDefinitionRegistry.registerBeanDefinition("Animal", beanDefinition);
    }
}
```

2. 使用实例：

   ```java
   // 待注入的Animal
   public class Animal {
   }
   
   // 配置类
   @Configuration
   //使用@Import导入组件，ID看beanDefinitionRegistry中注入时的ID名
   //如上面使用beanDefinitionRegistry.registerBeanDefinition("Animal", beanDefinition)，则容器中Animal类实例的ID为"Animal"
   @Import({MyImportBeanDefinitionRegistrar.class})     
   public class AppConfig {
   }
   ```

3. 理解：主要通过调用`BeanDefinitionRegistry`将类实例注入到容器中。

4. **这里`@Import`的value属性直接使用`ImportBeanDefinitionRegistrar`实现类的class对象即可**。

 ## 参考

1. @Import注解 -【Spring底层原理】：https://juejin.cn/post/6934499568037937160