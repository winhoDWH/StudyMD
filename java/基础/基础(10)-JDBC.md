### SQL基础
1. DML语句（Data Manipulation language数据操作语句）：主要由`insert`、`update`和`delete`三个关键词完成；**执行结果为表中多少行受影响**
2. DDL语句（Data Definition language数据定义语句）：主要由`create`、`alter`、`drop`和`truncate`四个关键词完成；
3. DCL语句（Data Control language数据控制语句）：主要由`grant`和`revoke`两个关键字组成；
4. 事务控制语句：主要由`commit`、`rollback`和`savepoint`三个关键字完成；
5. `select`语句

### JDBC
#### 连接基本流程
获取数据库驱动-->获取数据库连接-->获取对话-->通过对话执行sql语句；
```java
public static void main(String[] args) throws Exception {
        /**第一步：获取驱动**/
        Class.forName("com.mysql.jdbc.Driver");
        //可以使用try语句关闭资源
        try (
                /**第二步：获取链接**/
                //url：通常遵循这样写法：jdbc:[数据库驱动]:other stuff(按不同数据库决定)
                //mysql为：jdbc:mysql:ip:port/databaseName
                Connection connection = DriverManager.getConnection("jdbc:mysql://192.168.126.3", "root", "123456");

                /**第三步：获取Statement对象**/
                Statement statement = connection.createStatement();

                /**第四步：通过statement对象执行sql语句获取结果集**/
                ResultSet resultSet = statement.executeQuery("select * from user");
                )
        {
            /**第五步：操作结果集获取结果信息**/
            while (resultSet.next()){
                System.out.println(resultSet.getInt(0) + "\t"
                        + resultSet.getString(1));
            }
        }
    }
```

#### 关键类展示
##### `DriverManager`
1. 作用：获取数据库连接
2. public static Connection getConnection(String url,String user, String password) throws SQLException 

##### `Connection`
1. 作用：代表数据库连接对象，可设置连接属性，一个实例代表一个数据库物理连接会话，主要用来获取执行数据库语句的工具`Statement`;
2. 常用：
    * `Statement createStatement()`：获取一个工具对象Statement执行sql；
    * `PreparedStatement prepareStatement(String sql)`:获取一个**预编译**的工具对象执行sql；
    * `CallableStatement prepareCall(String sql)`:获取一个可用于执行调用存储过程的工具对象；
3. 控制事务的方法：
    * 设置保存点方法，会返回一个`Savepoint`保存点对象；
    * 回滚事务：通过`Savepoint`对象，或者默认回滚到上一个保存点；
    * 提交事务：commit()
    * 设置隔离级别

##### `Statement`
1. 作用：用于执行sql语句
2. 有三种执行sql的方式：
    * 第一种：执行查询语句。使用`ResultSet executeQuery(String sql)`;
    * 第二种：执行DML或者DDL语句。使用`int executeUpdate(String sql)`;DML语句返回受影响行数；DDL语句返回0；
    * 第三种：执行任意sql语句。使用`boolean execute(String sql)`;比较特殊，如果**执行结果为ResultSet**则返回`true`；**执行没有结果或者为受影响行数**，则为`false`；
    * 使用`execute()`方法执行任意sql语句后，如果要获取**结果集**（即返回`true`）则使用`statement.getResultSet()`
    * 如果想要获取受影响行数（即返回`false`），则使用`statement.getUpdateCount()`;

##### `PreparedStatement`
1. 作用：存放预编译的sql语句，通常用于可输入参数查询情况；
2. 方法：`setXXX(int index, XXX param)`,通过语句下标设置查询参数；
3. 如果不知道查询参数的类型，可以使用`setObject(int index, Object param)`方法，即让`PreparedStatement`自动进行类型转换；
4. 优势：无需拼接SQL；防止SQL注入；

##### `ResultSet`
1. 作用：查询结果集对象；可通过索引或者列名获取数据值；
2. 提供了很多方法去调整记录指针，用来获取不同行的数据（即List<Map<String, Object>>结果集中某个Map）；
3. 可使用`getXXX(int index)`或者`getXXX(String name)`获取数据对象；最通用的就是`getString()`，因为除了**Blob类型外**，所有类型都可以转换成String类型；
