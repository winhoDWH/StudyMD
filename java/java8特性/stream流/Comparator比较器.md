## Comparator接口

1. 作用：一般用于`stream`流的`sort()`方法中自定义排序标准；
2. 实现该接口需要实现`int compare(T o1, T o2);`

## 实现Comparator接口常用方法

1. 使用`Comparator`一个静态方法，完整定义为：` public static Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor, ....) `;
1. 其源码为：

```java
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            //这里调用了keyExtractor方法返回值的compareTo()方法
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
```

3. 根据上述源码可知，我们需要实现的`keyExtractor`参数（需要实现`R apply(T t);`方法，其含义为将某个对象转换成另一种类型）返回的值需要实现`compareTo()`方法。

4. 示例：

```java
//简单示例
Stream.of(1, 2, 3, 4).sorted(Comparator.comparing(i -> i)).toArray();
```

- 示例中，流中元素为`Integer`类型，自然实现了`compareTo()`方法，所以可直接用。

5. 另外有重载方法：

   ```java
   //多一个参数keyComparator，即可以指定选中转换后元素排序方法。
   public static <T, U> Comparator<T> comparing(
           Function<? super T, ? extends U> keyExtractor,
           Comparator<? super U> keyComparator)
   {}
   ```

## `thenComparing()`方法

1. 该方法为**实例方法**，一般用于多标准排序的时候，表示**再根据什么进行排序**；
2. 例如：

```
Stream.of(1, 2, 3, 4).sorted(Comparator.comparing(A).thenComparing(B))
```

- 上诉示例功能为：先按A进行排序，后按B进行排序；

## 倒序

1. 使用**实例方法**，`reversed()`方法
2. 示例：`Comparator.comparing(A).reversed()`;

