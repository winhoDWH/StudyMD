## spring组成
<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/a8c8f894-a712-447c-9906-5caef6a016e3.png"/> </div><br>

1. spring core:核心基础
2. spring AOP：
3. spring ORM: 对mybatics等框架的支持；
4. spring DAO: 对dao层的支持；
5. 

## 扩展
1. 用`springboot`构造一切(build anything)；用`spring cloud`协调一切(coordinate anything);用`srping cloud data flow`连接一切（connect anything）
2. `springboot`是一个快速开发的框架，快速开发一个微服务；**约定大于配置**
3. `springcloud`是基于`springboot`实现的;

## IOC（控制反转）
1. 简单理解：通过Set方法，实现一个类里面对象的动态设置，使得程序的耦合性大大降低；**原来对象的创建完全由程序进行控制，但IOC是将对象的创建交给用户，由用户创建之后再赋给对象（即将创建对象交给第三方）；**获取对象的方法反转了。
2. **DI(依赖注入)是IOC的一种方法**；可以使用XML文件配置实现IOC，也可以使用注解等方式实现；
3. spring容器会在初始化时先读取配置文件，根据配置文件创建或组织对象放入`IOC容器`中，程序使用时从容器中取出；
4. **总结标准答案**：IOC是一个根据描述(XML文件)，通过第三方创建或获取对象的方式，在spring中实现IOC控制反转的是IOC容器，其实现方法是依赖注入（Dependency Injection DI）