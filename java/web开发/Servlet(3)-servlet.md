### Servlet
#### 生命周期
1. 只有两个状态：不存在和初始化；**注意这里的初始化不是类初始化中的创建实例，初始化成员变量的过程，而是已经创建好实例，调用该实例的service()方法去处理请求的过程，即服务器处理对应请求时的运行全过程**
2. 不存在->初始化状态：容器加载对应Servlet类，初始化servlet（运行构造函数），调用Init()方法创建Servlet实例（**该方法Servlet一生只调用一次**）
3. 初始化状态：被容器调用service()方法在单独线程中处理各种请求；
4. 初始化->不存在：容器调用destroy()方法，杀死servlet实例；
5. init()方法可以被覆盖，用于做一些数据库连接等初始化操作，如果不重写该方法则调用GenericServlet类中的init()方法；而service()方法一般不重写，因为这是用来判断HTTP请求方法的，一般重写doGet和doPost方法；
6. 一个Servlet在一个容器中是**单例**的，容器中会对每一个请求创建或分配一个线程去执行Servlet的service()方法；
7. 容器在启动时就去寻找已经部署的WEB应用，然后寻找对应的servlet类文件；找到类文件之后，加载类的工作可能是启动时或者在第一个请求过来时进行，但**init()方法一定在service()方法被调用前执行，即提供请求服务前Servlet一定是初始化状态**；

#### 初始化
1. Servlet在进入初始化状态前，会被调用构造函数和init()方法，而调用构造函数**只是让其成为一个对象，而不是Servlet**,调用init方法后，该对象才有Servlet该有的特性；
2. 特性：使用ServletConfig对象和ServletContext对象
3. 当容器初始化一个servlet的时候，会为这servlet建一个唯一的ServletConfig对象；**容器会从DD（web.xml）中读取servlet初始化参数，然后将这些参数交给ServletConfig对象，再将这个对象在init（）方法时传递给servlet**；
4. 

##### ServletConfig
1. 每个Servlet都有一个ServletConfig对象，用于**向Servlet传递部署时信息（配置文件信息）和访问ServletContext**
2. **每一个Servlet的init()方法都有两种，一是无参的，二是有一个ServletConfig作为参数的；一般重写是重写无参的，因为有参数的会调用无参的init()方法，而容器调用有参数的init()方法进行初始化**；
3. servlet拥有一个getServletConfig()方法获取ServletConfig对象，但注意**在容器调用了init()方法后，才拥有ServletConfig对象，所以不要在调用init()方法，获取ServletConfig对象进行操作**；
4. ServletConfig对象有一个**getInitParameter("String")**方法获取对应配置文件中的属性值；
5. web.xml配置文件中写：
    ```
        <servlet>
            <servlet-name>s1</servlet-name> //内部名，用来做url与class路径之间的映射
            <servlet-class>com.example.xxxServlet</servlet-class> //servlet类的路径

            <init-param>
                <param-name>String</param-name> //对应上面getInitParameter("String")
                <param-value>123</param-value>
            </init-param>
        </servlet>
    ```
    * 注意：**在对应servlet声明标签<servlet>中使用<init-param>来声明属性**
6. web.xml文件只在容器初始化servlet的时候才会被读一次，如果修改了则必须要重启web应用；
7. 其他方法：
    * getServletContext()：获取上下文；
    * getServletName():获取servlet名？
    * Enumeration getInitParameterNames()：获取属性名组；返回是个Enumeration对象，通过其nextElement()方法获取属性名；




##### ServletContext
1. **每个应用都有一个ServletContext(上下文)**；用于访问WEB应用中通用的参数（所有Servlet都可以获取到）和服务器信息；
2. web.xml中声明：
    ```
    <context-param>
        <param-name>password</param-name> 
        <param-value>123</param-value>
    </context-param>
    ```
    * 注意：由于是面向所有servlet的属性设置，所以该标签不被包含在任何servlet标签中，而是在<web-app>这个web.xml文件总体标签中；
    * config中使用的是<init-param>标签，而Context中使用的是<context-param>标签
    * **config的属性和context的属性名是可以相同的，因为调用不同的对象去获取，所以就没有冲突！！！**
3. 代码中,类似config对象的方法，调用ServletContext中的getInitParameter("String")方法即可获得对应属性值；
4. ServletContext还可以作为servlet之间通信的桥梁；
5. 获取ServletContext对象的方法
    * **在servlet类中调用getServletContext()方法**获取一个ServletContext对象；
    * 如果是在servlet类以外的类想获取ServletContext对象的话，**传入一个ServletConfig对象，调用其getServletContext()方法即可**；


### 监听者
1. 需求：在所有servlet初始化之前做一些操作，让所有servlet都可以使用；
2. ServletContextListener能监听ServletContext一生中关键的两个时间：初始化和撤销；
3. 实现该ServletContextListener接口的类，其对象可以做到：
    * 在上下文ServletContext初始化时得到通知；一从ServletContext得到初始化参数；二是创建一个对象，使web引用所有部分都可以访问；
    * 上下文撤销（web应用结束）是得到通知，就可以做一些比如关闭数据库连接的操作；
```

```



在Servlet中（web应用中）使用I/O填写文件路径时，"/xxxx.txt"代表在**对应web应用根目录（其根目录下还有WEB-INT文件）**下查找xxxx.txt文件；

### HttpServletResponse和HttpServletRequest
1. 两个只是接口类，其具体实现是由容器开发商实现的，平时在业务代码中获取到的实例都是容器创建好的，所以这两个实际上只是一个规范，API；
2. HttpServletResponse的setContentType()方法设定返回内容类型，使用的原因：**服务器**不能通过返回内容中的文件的后缀名来判断文件类型，只有通过该属性才能通知服务器Servlet返回的内容是什么类型的，从而让服务器**设置HTTP响应中的内容类型属性**；
3. 请求重定向：调用setRedirect()方法，**浏览器做的事情**
    * 过程：servlet在使用该方法后，服务器会将HTTP响应的状态码修改为**301**，还有添加**一个Location的首部属性**，该属性的值就是重定向的URL；浏览器获取到响应后，针对状态码会找到Location首部属性并建立一个新的请求，**此时浏览器url地址上就会变成新的url地址**；
    * 在setRedirect()方法中填入URL作为参数，该url有三种情况：
        1. 采用绝对路径url，即：http://ip:port/xxxxx
        2. 采用相对路径，相对于当前web应用（**填入的url不以/开头**），如：重定向前原本的url：http://ip:port/web/s1/hello.html，请求S1应用中的hello.html，重定向时使用setRedirect("foo/word.html"),则重定向后的url为http://ip:port/web/s1/foo/word.html；**所以是相对于当前应用**
        3. 采用相对路径，相对于**服务器根目录（填入URL以/开头）**，如：重定向前原本的url：http://ip:port/web/s1/hello.html，重定向时使用setRedirect("/foo/word.html"),则重定向后的url为http://ip:port/foo/word.html;**开头/代表相对于web容器根目录**
4. 请求分派：**服务器做的事情**
    * 使用请求分派：url不会发生改变，客户端并不知道请求处理的servlet发生改变；
    ```
        RequestDispatcher view = request.getRequestDispatcher("url");
        view.forward(request, response);
    ```