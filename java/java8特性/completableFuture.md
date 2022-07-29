## CompletableFuture 

1. 作用：对Future模式的增强，针对FutureTask（异步任务）执行结束后，回调获取任务执行结果的处理，从原本get()阻塞主线程（控制流仍在主线程），提供另外的线程去执行回调处理操作。
2. 



## 扩展--Netty工具类中扩展Future

1. Netty框架中扩展了原生的Future类，使有更多功能。
2. 依赖

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.29.Final</version>
</dependency>
```

3. 使用示例：

```JAVA
import io.netty.util.concurrent.DefaultEventExecutorGroup;
import io.netty.util.concurrent.EventExecutorGroup;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.FutureListener;

public static void main(String[] args) throws InterruptedException {
    EventExecutorGroup groups = new DefaultEventExecutorGroup(4);
    System.out.println("开始：" + getDateStr());

    //工作线程
    Future<Integer> f = groups.submit(new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            System.out.println("开始计算：" + getDateStr());
            Thread.sleep(1000);
            System.out.println("计算结束" + getDateStr());
            return 100;
        }
    });

    //添加线程观察者
    f.addListener(new FutureListener<Object>(){

        @Override
        public void operationComplete(Future<Object> objectFuture) throws Exception {
            System.out.println("计算结果：" + objectFuture.get());
        }
    });

    System.out.println("结束：" + getDateStr());
    //不让守护线程退出
    new CountDownLatch(1).await();
}
```

4. 代码以及类说明：
   1. 注意这里的`Future`使用的是Netty自己封装的Future类，继承了java原生的Future，扩展了更多地方法提供使用。
   2. `EventExecutorGroup`类则是实现了JDK中`ScheduledExecutorService `接口的线程组接口，所以它拥有线程池的所有方法。其中示例里使用的submit()方法，也是`EventExecutorGroup`自己重写，将原本返回的JDK原生`Future`类，改成自己封装的Future类；
   3. addListener()方法的作用，提供一个回调方法。

## 参考

https://www.jianshu.com/p/0d0d3054e108