1. 在openJDK官网或者github上面找到如jdk7u-jdk的jdk项目；
2. 在该项目中搜索对应的native方法

例如这里我们查找`Thread.interrupt`方法

![image-20221128110225501](img/image-20221128110225501.png)

![image-20221128105827572](img/image-20221128105827572.png)

![image-20221128105903255](img/image-20221128105903255.png)

比如我这里想搜索tread的执行源码，则找到对应`.c`文件

![image-20221128110042336](img/image-20221128110042336.png)

找到对应C语言的方法之后，可以去`jdk8u_hotspot`等jdk_hotspot项目中查找对应的

![image-20221128112617646](img/image-20221128112617646.png)

![image-20221128112640736](img/image-20221128112640736.png)

找到当中的`jvm.cpp`文件查找对应的方法