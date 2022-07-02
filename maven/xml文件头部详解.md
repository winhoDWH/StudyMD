## 详解xml文件中xmlns:xsi , xmlns, xsi:schmeLocation 三个属性的作用
### 示例
```
<!--以maven的pom.xml文件头部为例-->
<project 
xmlns="http://maven.apache.org/POM/4.0.0" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
```

### xmlns属性
1. 首先当多个xml文件或者单个xml文件中出现多个**名称相同但定义不同**的元素时，xml解析器是无法解析的，无法分辨出使用具体哪一个元素；如：
    ```
        <!--下面两个table元素定义不同-->
        <!-- 这里的 table 元素描述的是一个表格-->
        <table>
        <tr>
        <td>Apples</td>
        <td>Bananas</td>
        </tr>
        </table>
        <!-- 这里的 table 元素描述的是一个家居桌子-->
        <table>
        <name>African Coffee Table</name>
        <width>80</width>
        <length>120</length>
        </table>
    ```
2. 使用前缀可以简单解决冲突问题，如：
    ```
        <!-- 这里的 table 元素描述的是一个表格-->
        <h:table>  <!--添加了前缀 h -->
        <h:tr>
        <h:td>Apples</h:td>
        <h:td>Bananas</h:td>
        </h:tr>
        </h:table>
        <!-- 这里的 table 元素描述的是一个表格-->
        <f:table> <!--添加了前缀 f -->
        <f:name>African Coffee Table</f:name>
        <f:width>80</f:width>
        <f:length>120</f:length>
        </f:table>
    ```
3. 但是xml文件之间可以通过XInclude, External Entites 实现相互包含或者引用的。这样前缀就不一定能完全避免冲突问题；而且每个元素都写上前缀也比较麻烦；于是**命名空间这一概念**就诞生了；
4. 命名空间
    * 给xml元素定义一个很长且唯一的字符串，于是就使用URL（统一资源标识符）来确保唯一性；实际上很多公司都会将这个URL指向包含很多信息的网页等；
    * 使用命名空间后，可以使用**默认命名空间的写法**，实际上就是平常写xml文件的方式，不需要为每个元素添加前缀（这就是为什么不怎么见使用前缀的原因）；
5. **xmlns的值，实际上就是一个命名空间，保证一个xml文件的唯一性**；

### xsi:schemaLocation属性
1. 该属性实际上是**一个以空格为分隔符的键值对形式**；如例子中，键key为：http://maven.apache.org/POM/4.0.0，值values为：https://maven.apache.org/xsd/maven-4.0.0.xsd；
2. 其值values对应的，是一个XSD文件（XML Schema Definition，xml模式定义文件）的位置；XML解析器根据这个XSD文件的内容来解析这个XML文件，判断XML文件中的结构是否符合标准；**所以XSD文件就是XML文件的语法检查器**；在官网上可以找到产品对应版本的XSD文件；

### xmlns:xsi属性
1. 该属性实际上是声明**前缀xsi的**一个命名空间，是**xsi:schemaLocation属性中指向的XSD文件的命名空间**；（自己可以点开链接看一下，对应XSD文件中的xmlns:xs）
2. 业界默认都使用**http://www.w3.org/2001/XMLSchema-instance**作为该属性的值，即几乎所有的XSD文件都用这个字符串作为命名空间；

### XSD文件
XSD文件中有一个属性targetNamespace，实际上**规定了我们自己编写的XML文件中xmlns属性的值，即默认命名空间**；

### 参考
https://blog.csdn.net/lengxiao1993/article/details/77914155  详解 xml 文件头部的 xmlns:xsi