### Lambda表达式
1. 用途：简化匿名内部类的写法；实际上就是使用一个匿名内部类**创建一个对象！**；
2. **表达式的目标类型是函数式接口（只有一个抽象方法的接口）**,即Lambda表达式只能实现一个方法；
3. 语法定义：(参数列表)->{方法体}；参数列表是实现的接口的方法的参数列表，**可以省略参数类型**，如：
```
interface Table{
    //一个抽象方法，参数列表为
    void t(int a,int b);
}
class Test(){
    //参数为一个接口的方法
    void test(Table table){
        table.t()
    }
    void test2(){
        //调用test()方法，使用lambda表达式
        test((a,b)->{
            System.out.println(a+b);
        });
    }
}
```
4. 函数式接口返回的就是一个对象，所以可以用来赋予一个对象的值；如：
```
Runnable r = () ->{
    for(int i = 0; i < 10; i++){
        System.out.println(i);
    }
}
```
5. 当Lambda表达式的代码块中只有一行代码时，且这行代码是调用别的类的类方法、实例方法、构造器等时，可以通过方法引用与构造器引用来简化代码（具体看表）。
6. Lambda表达式不能调用函数式接口中的**默认方法**；


###  @FunctionalInterface函数式接口注解

### java.util.function包下函数式接口规则（可用来作为自己写代码的参考）
1. XXXFuntion接口：通常包含一个apply()抽象方法，用来对参数进行处理、转换然后**返回一个新的值**；**通常该类接口用来对数据进行转换处理**
2. XXXConsumer接口：通常包含一个accept()抽象方法，与上面apply()类似，也对数据进行处理，但**没有返回值**；
3. XXXPredicate接口：通常包含一个test()抽象方法，**对参数进行判断，然后返回boolean值**；**常用来筛选数据**
4. XXXSupplier接口：宝行一个getAsXXX()方法，**不输入参数**，但会返回一个数据；