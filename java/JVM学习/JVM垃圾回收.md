## JVM垃圾回收

### 类的实例存放方式

1. 年轻代、老年代、永久代
2. `永久代`：就是方法区、存放加载之后的类的信息；

### 扩展知识

1. 方法区进行垃圾回收的条件：

   > 1. 该类的所有实例对象都被垃圾回收；
   > 2. 该类的类加载器`ClassLoader`已被回收；
   > 3. 对该类的`Class`对象没有任何引用；

2. 