### stream
1. stream()是java8新增的类，用来补充集合类；
2. stream是一种数据流的方式，提供了**串行和并行两种类型的流**；

#### 特性
1. 不存放数据
2. 不修改数据源（函数式编程）
3. 延迟操作
4. 解绑，即可以限制操作多少个数据
5. 纯消费：每个流中的元素只访问一次；

#### 创建流
1. Collection集合创建流：stream()和parallelStream()两种;并行流和串行流简单区别：并行流是**内部以多线程**执行操作，前提是流中数据处理**没有顺序要求**；顺序流可以通过`parallel()方法`修改为并行流
    ```
        List<String> list = new ArrayList<>();
        //串行流
        Stream<String> listStream = list.stream();
        Stream<String> listToPaStream = listStream.parallel();
        //并行流
        Stream<String> paStream = list.parallelStream();
    ```
2. Array数组使用`Arrays工具类`中的stream()方法：`Arrays.stream(nums)`;
3. `Steam类`中的静态方法：of()、iterate()、generate()
    * `of()`：入参为一个数组/多参数(T ...a)或者一个数值(T a)；
    * `iterate(final T seed, final UnaryOperator<T> f)`：根据第一个参数`seed`放入到函数f中生成**一组无限按函数f规律递归的数据的stream流**
    * `generate(Supplier<T> s)`:生成一组按s函数规则的**无限stream流**；一般函数s是**生成随机数或者恒定数值**
    ```
        //iterate方法,返回一个无限流
        Stream<Integer> itStream = Stream.iterate(2, n -> 2+n).limit(6);
        itStream.forEach(n -> System.out.println(n));
        //输出结果为：2,4,6,8,10,12

        //generate方法，返回一个无限流
        //下面生成结果为一组随机数
        Stream<Double> genStream = Stream.generate(Math::random).limit(6);
    ```
4. `BufferedReader.lines()`方法，将每行内容转成流
    ```
        BufferedReader reader = new BufferedReader(new FileReader("F:\\test_stream.txt"));
        Stream<String> lineStream = reader.lines();
        lineStream.forEach(System.out::println);
    ```
5. `Pattern.splitAsStream()` 方法，将字符串分隔成流
    ```
        Pattern pattern = Pattern.compile(",");
        Stream<String> patStream = pattern.splitAsStream("a,b,c,d");
        stringStream.forEach(System.out::println);
    ```


#### 流的中间操作
##### 筛选和切片
1. `filter(Predicate<? super T> predicate)`:过滤,参数是一个函数式接口；
2. `limit(n)`:获取前n个元素，**如果n大于流中元素个数，则表示取所有元素，并不报错或补默认值**
3. `skip(n)`:跳过前n个元素，配合limit(n)可实现分页；**下标从0开始计算**
4. `distinct`:通过流中元素的hasCode()和equals()去重（**需要满足两个方法都相等才认为两元素相同**）

##### 映射
1. `map(funtion)`:接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素；
2. `flatMap(funtion)`：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流， 
3. 两者区别：`map()`适用于**将流中每个元素转变成另一种类型的元素**的场景，例如将List<Integer>转变成List<String>；`flatMap()`适用于**将流中每个元素都进行拆分并汇聚成新流**场景，如：一个集合为：`{'1,2,3', '4,5,6'}`这种，元素可以根据逗号拆分的；


##### 排序
1. `sorted()`:自然排序，流中元素需要是吸纳Comparable接口；
2. `sorted(Comparator com)`：定制排序，自定义Comparator排序器；

##### 消费
1. `peek(funtion)`：类似于map，都是用于**转换每个流中元素**，但是其参数函数式接口中，需要实现的方法是没有返回值的，即写入的lambda表达式没有返回值；而map是有返回值的

##### 过滤
1. `filter`:不符合的数据将会从流中去掉

#### 流的终止操作
1. allmatch，noneMatch，anyMatch用于对集合中对象的某一个属性值进行判断，传入一个参数，该参数说明判断条件；
    * allMatch全部符合该条件返回true，**都匹配**
    * noneMatch全部不符合该断言返回true，**都不匹配**
    * anyMatch 任意一个元素符合该断言返回true，**部分匹配**

2. **获取流中某一元素或值**
    * findFirst：返回流中第一个元素，无参
    * findAny：返回流中任意元素，无参
    * count：返回流中元素的总个数，参数为**判断元素之间大小的函数**
    * max：返回流中元素最大值，参数为**判断元素之间大小的函数**
    * min：返回流中元素最小值，参数为**判断元素之间大小的函数**
    * **注意上面返回的都是一个Optional类型,通过调用其get()方法即可获取其对应值**

3. reduce：归约，将一个流缩减成一个值，能实现对集合求和、求乘积和求最值操作；count、min、max底层就是使用reduce。
4. toArray()：默认不传参数，转化成`Object[]`;
5. collect:收集，主要依赖collectors的静态方法

### Optional类
1. Optional类是一个可以为null的容器对象。如果值存在则`isPresent()`方法会返回true，调用`get()`方法会返回该对象。(**无值时使用get方法会返回什么**)
2. `Stream`中的元素是以`Optional类型`存在的;

### 参考
1. Java Steam详解：https://blog.csdn.net/qq_37126175/article/details/106993656
2. https://blog.csdn.net/mu_wind/article/details/109516995