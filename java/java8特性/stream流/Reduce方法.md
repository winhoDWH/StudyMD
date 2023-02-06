## Reduce方法

1. 作用：将流中的数据按某种规则进行合并从而获取到一个值；

2. 有三种重载方法，三个参数；

3. 当并行使用reduce方法时，要注意一下几点：
   - 结果和处理的顺序无关
   - 操作不影响原有数据
   - 操作没有状态和同样的输入有一样的输出结果

## 一个参数

1. `Optional reduce(BinaryOperator accumulator);`
2. 这个参数的名称accumulator（累加器），它的类型是一个函数式接口;该接口实现需要传入两个参数。
3. 实例：

```java
// 这里的acc+i的值都是下一次循环的acc
Optional<Integer> sum = Stream.of(1, 2, 3).reduce((acc, i) -> acc + i);
// 6
System.err.println(sum.get()); 
```

4. 如上示例：实现累加器的方法的两个参数：`acc`是流中数据一路累加的值，`i`是当前计算到的流中数据；

## 两个参数

1. ###### `T reduce(T identity, BinaryOperator accumulator);`

2. `identity`参数是一个初始值，理解成进行累加器`accumulator`数据处理前的初始值；
3. **直接返回第二参数方法泛型类型的值**；
4. 实例：

```java
// 这里的acc+i的值都是下一次循环的acc
Integer sum = Stream.of(1, 2, 3).reduce(Integer.valueOf(1), (acc, i) -> acc + i);
// 7   初始化值是1
System.err.println(sum);
```



## 三个参数（并行流）

1. `U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator combiner）`
2. `combiner`参数是用来合并的，主要是用于当流是`并行流`的时候使用该三个参数的方法。
3. 在使用该重载方法时，`identity`值是都分配给每一个流操作的线程，需要注意该对象并发操作的问题。(**上面两个参数的重载方法同样也会有一样的问题**)

```java
String sum = Stream.of(1, 2, 3).parallel().reduce(String.valueOf("1"),
                (acc, i) -> String.valueOf(Integer.valueOf(acc) + i),
                (acc1, acc2) -> String.valueOf(Integer.valueOf(acc1) + Integer.valueOf(acc2)));
// 9 把上面的的那个案例有单行流改成了并行流，这个结果是不一定的，可能8可能7，看并行流数量
System.err.println(sum);
```

4. `combiner`接口是两个参数，代表两两线程的值合并。



## 参考

原文链接：https://blog.csdn.net/Lou_Lan/article/details/120667822