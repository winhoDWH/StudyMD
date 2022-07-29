## 源码跟踪

#### 1. header标题初始化的源码跟踪比较复杂，因为原本有很多builder，而且实际读写设置实体类参数都是继承同一个`Param`类的，所以下面只取读的时候初始化场景，可能有误；

#### 2. 从sheet方法进入

![image-20220720173754181](img/image-20220720173754181.png)

![image-20220720173810754](img/image-20220720173810754.png)

![image-20220720173829026](img/image-20220720173829026.png)

![image-20220720173846200](img/image-20220720173846200.png)

![image-20220720173932071](img/image-20220720173932071.png)

![image-20220720173959197](img/image-20220720173959197.png)

![image-20220720174102342](img/image-20220720174102342.png)

![image-20220720174118054](img/image-20220720174118054.png)

![image-20220720174146115](img/image-20220720174146115.png)

直到这里，我们能找到`ExcelReadHeadProperty`初始化的代码，因为这个类的实例就是程序匹配excel标题时用到的。

#### 3. 进入到`ExcelHeadProperty`类的构造方法（读写都是继承该类的）

![image-20220720174422896](img/image-20220720174422896.png)

注意这里`headKind`，如果你是通过字符串数组的方式设置header（标题）的话，则这里的headKind就是`HeadKindEnum.STRING`；如果是通过实体类，则走`initColumnProperties(holder, convertAllFiled);`方法

#### 4. 看具体如何解析实体类（`@ExcelProperty`注解）

![image-20220720174643598](img/image-20220720174643598.png)

主要看这两个方法，首先说明一下参数：

- `sortedAllFiledMap`：是最终排序好的标题与实体类中`File`对应；
- `indexFiledMap`：是注解中加了index属性的File集合；
- `ignoreMap`：应该是`@ExcelIgnore`标签的File集合（没具体研究了）。

以上参数都在`ClassUtils.declaredFields`方法中解析后赋值；

主要看`ClassUtils.declaredFields`方法，主要跟踪Clazz对象：

![image-20220720175208399](img/image-20220720175208399.png)

![image-20220720175232175](img/image-20220720175232175.png)

![image-20220720175246316](img/image-20220720175246316.png)

#### 5. 上面遍历其导入类中File，调用`declaredOneField`方法，解析导入类中easyexcel注解内容。

![image-20220720175633837](img/image-20220720175633837.png)

#### 6. 解析完之后会进行对上面map进行排序，即对标题进行排序。

![image-20220720175913584](img/image-20220720175913584.png)



![image-20220720180345328](img/image-20220720180345328.png)

#### 7. 由上面源码可知，正如官网介绍的，`@ExcelProperty`标签中index和order是不能同时定义，index属性会把order属性无效化。