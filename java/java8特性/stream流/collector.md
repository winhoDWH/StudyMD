## 1.	Collector收集器-接口
1. 原理：是一个可变的**汇聚操作**，将流中元素汇聚到一个可变新容器中，待全部操作处理完成后，将累计结果转换成最终的表示（这一步转换是可选的，当最终结果类型和中间操作处理后的元素类型不一致时才进行）；支持串行和并行；
2. `public interface Collector<T, A, R>`,三个元素分别代表含义：
    * T：输入的元素的类型；
    * A：累计（处理过程中）结果容器的元素类型；
    * R：最终结果类型；
3. 通过四种方法完成汇聚操作：
    * supplier： 创建新的结果容器
    * accumulator：将输入元素合并到结果容器中
    * combiner：合并两个结果容器（并行流使用，将多个线程产生的结果容器合并）
    * finisher：将结果容器转换成最终的表示

## 2.	Collectors类-Collector实现类
1. 提供常见的Collector类实现（具体是定义了**实现Collector接口的内部类**CollectImpl），Collects工具类所有静态方法返回都是CollectImpl对象；
2. 下面介绍不同Collects中的静态方法：

#### 2.1	求均值：`averagingXXX`
1. 分别有三种求平均值的方法：`Collectors.averagingInt(x)`,`Collectors.averagingLong(x)`以及`Collectors.averagingDouble(x)`;**注意这里`averaging`后面的类型是根据用来统计计算的元素类型决定，返回都是Double类型的平均值结果**
2. 三种方法的参数`x`实际上是**说明怎么获取用于计算平均值的值**；如：
```
    List<Apple> data = Arrays.asList(new Apple("green", 210), 
			new Apple("red", 170), new Apple("green", 100), new Apple("red", 170), 
			new Apple("yellow", 170), new Apple("green", 150));
    //由于用于计算的值为Int类型
	//这里Apple::getWeight方法是说明获取上面data中元素的中weight属性用来求平均值的属性，返回值为Int类型
    //注意返回值为Double
	Double result = data.stream().collect(Collectors.averagingInt(Apple::getWeight));
```

#### 2.2	求总数，`counting()`
1. 与`Stream().count`有相同功能，都返回`Long类型`结果；但这里返回值为CollectorImpl类型的值，只不过`stream().collect()`将结果转为`Long`类型；
2. 直接使用`Collectors.counting()`即可；

#### 2.3	求多类值（包含和、平均值、最大最小值、元素个数）`summarizingXXX()`
1. 分别有：`Collectors.summarizingInt(x)`,`Collectors.summarizingLong(x)`、`Collectors.summarizingDouble(x)`；
2. 同`averagingxxx()`后面的类型根据具体参与计算的数据类型决定；
3. 同`averagingXXX()`的参数，都是**根据用于计算的值的类型决定使用什么方法**；其经过`stream().collect()`处理后返回值分别为：`IntSummaryStatistics`，`LongSummaryStatistics`,`DoubleSummaryStatistics`;
```
    IntSummaryStatistics summaryStatistics = data.stream().collect(Collectors.summarizingInt(Apple::getWeight));
    //输出IntSummaryStatistics{count=6, sum=970, min=100, average=161.666667, max=210}
    System.out.println("求多类值summarizingInt结果：" + summaryStatistics);
```

#### 2.4	分组
1. `groupingBy()`,有多个参数的重载，适用场景：根据什么值对流进行分组
    * 单个参数：`groupingBy(Function<? super T, ? extends K> classifier)`：classifier决定结果Map（HashMap）的键
    * 两个参数：`groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)`：downstream决定结果Map的值，**单个参数中是以List集合作为Map的值**;上面提到的`counting`返回值为collector实现的就可以作为该参数；（同理还有最大最小值、平均值，总数）
    * 三个参数： `groupingBy(Function<? super T, ? extends K> classifier, Supplier<M> mapFactory,  Collector<? super T, A, D> downstream)`：  mapFactory指定结果Map的类型
```
        //单个参数
        Map<String, List<Apple>> groupCollector1 = data.stream().collect(Collectors.groupingBy(Apple::getName));
        //两个参数
        Map<String, Long> groupCollector2 = data.stream().collect(Collectors.groupingBy(Apple::getName, Collectors.counting()));
        //三个参数
        TreeMap<String, Long> groupCollector3 = data.stream().collect(Collectors.groupingBy(Apple::getName, TreeMap::new,Collectors.counting()));
```
2. 类似的还有`groupingByConcurrent()`方法，该方法将元素整理成`ConcurrentMap`；

#### 2.5	分区
1. `partitioningBy`,是分组的特殊情况，**将流中元素分成两类，即根据符不符合条件进行分组**；
    * 单个参数：`partitioningBy(Predicate<? super T> predicate)`：predicate提供分区依据;
    * 两个参数：`partitioningBy(Predicate<? super T> predicate,Collector<? super T, A, D> downstream)`：downstream提供结果Map的值

#### 2. 6	字符串操作`joining`
1. 对流中元素（**需要元素都为String类型**）进行拼接；
    * `joining()`：拼接输入元素
    * `joining(CharSequence delimiter)`：将delimiter作为分隔符
    * `joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)`：将prefix作为前缀，suffix作为后缀
```
    String result = data.stream()
                .map(Apple::getName).collect(Collectors.joining(",", "Color[", "]"));
    System.out.println(result);
```

#### 2.7	数据转换
1. `mapping(f1, f2)`：将流中元素先执行f1函数，在执行f2函数；

#### 2.8	转化成集合
1. `toCollection(Supplier<C> collectionFactory)`：将输入元素整理成集合，collectionFactory可以指定结果集合的类型
2. `toList()`：将输入元素整理成`ArrayList`;
3. `toSet()`：将输入元素整理成`HashSet`;
4. `toMap()`:整理成map；
    * `toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)`：keyMapper和valueMapper分别提供结果Map的键和值;**两个参数分别决定新Mapd的键和值**；
5. `toConcurrentMap()`操作与tomap相同；

### 参考
1. java8新特性(四) Collector（收集器）：https://blog.csdn.net/xiliunian/article/details/88773718?ops_request_misc=&request_id=&biz_id=102&utm_term=collector&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-88773718.pc_search_result_hbase_insert&spm=1018.2226.3001.4187