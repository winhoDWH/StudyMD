### setting.xml
1. <localRepository>标签：
    * 设置系统本地仓库路径，默认是{user.home}/.m2/repository/（**{user.home}是C:\Users\你的Windows账号该路径下**）
2. <interactiveMode>标签:
    * 指定Maven是否视图与用户交互获得输入，默认是true（实际运用在命令行使用maven指令的时候输入-xxx value这样的参数，很少用到）
3. <usePluginRegistry>标签：
    * 是否使用plugin-registry.xml文件来管理插件版本，默认为false。plugin-registry.xml文件内容看官方文档。
4. <offline>标签：
    * 编译或构建项目时是否连接远程中心仓库，默认为false；
5. <pluginGroups>标签：
    * 标签中可包含多个<pluginGroup>标签（该标签是pluginId列表），该标签的值代表一个groupId，如果我们在使用一个插件时没有声明其groupId，则会自动在该标签中查找一个groupId;
6. <proxies>标签：
    * **声明使用网络代理来连接远程库；**<proxies>标签下可以有多个<proxy>来设置代码需要的信息；
    * 默认使用第一个<proxy>标签设置的代理；
7. maven的**jar包先从本地仓库中查看，如果没有就去远程仓库找**；远程仓库有：中央仓库（maven团队自定义维护的仓库）和远程自定义仓库（自己搭建的）
8. <Servers>标签：
    * 连接远程仓库的设置，包含认证信息；
9. <mirrors>镜像
    * 作用：一般用来作为中央仓库的镜像，因为中央仓库下载太慢了，设置之后，在访问中央仓库时会先访问镜像的URL地址查找依赖包。

### 参考：
https://www.cnblogs.com/wdliu/p/8312543.html
