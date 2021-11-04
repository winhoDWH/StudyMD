## 配置标签讲解

### 一 import

1. 可以通过<import>将不同的人开发的配置文件合并成同一个，使用时直接使用总的配置即可;

2. 示例：

   ``` xml
   <import resource="bean1.xml"></import>
   ```

   

ClassPathXmlApplicationContext：类路径下的xml文件解析