## 使用目的
传统springboot项目maven打包是全量打包到一个jar文件中，然后使用java -jar启动；当我们只需修改某些配置文件或替换依赖包时，**我们需要重新进行打包操作，相当于全量更新**，十分不符合工程学原则，所以我们使用别的方式打包；

## 常见的打包工具
### maven-compiler-plugin插件
1. 编译java源码，一般只用来设置编译的jdk版本；
2. 第一种使用方法：在plugins标签汇中使用
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
    <version>
    <configuration>
        <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中不能使用低版本jdk中不支持的语法)，会存在target不同于source的情况 -->                    
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
    </configuration>
3. 第二种使用方法：在properties中设置jdk版本:
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

### maven-jar-plugin插件
1. 作用：打包（jar）插件，设定 MAINFEST .MF文件的参数，比如指定运行的Main class、将依赖的jar包加入classpath中等等，首先我们明确一点的是maven 插件功能：compile、package、deploy...都是在${project.build.directory }/classes 文件路径下，当然测试是在test-classes下；
2. 简易版
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.3.1</version>
    <configuration>
        <archive>
            <manifest>
                <!--运行jar包时运行的主类，要求类全名-->
                <mainClass>com.dwh.springdemo.SpringdemoApplication</mainClass>
                <!-- 是否指定项目classpath下的依赖 -->
                <addClasspath>true</addClasspath>
                <!-- 指定依赖的时候声明前缀 即设定classpath路径，./代表以当前项目文件为根目录-->
                <classpathPrefix>./lib/</classpathPrefix>
            </manifest>
        </archive>
        <!--希望过滤掉什么文件-->
        <excludes>
            <exclude>**/application*.yml</exclude>
            <exclude>**/log4j2*.xml</exclude>
            <exclude>**/generatorConfig.xml</exclude>
            <exclude>**/generator.properties</exclude>
            <exclude>**/*.sql</exclude>
        </excludes>
    </configuration>
</plugin>


### maven-assembly-plugin插件
插件依赖：
    ```
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <encoding>UTF-8</encoding>
                <!--当描述文件package.xml中使用includeBaseDirectory时才有用-->
                <finalName>${project.artifactId}</finalName>
                <!-- 包名不含assemblyId -->
                <appendAssemblyId>false</appendAssemblyId> 
                <skipAssembly>false</skipAssembly>
                <!--如果路径长度超过100字符,执行的操作warn" (default), "fail", "truncate", "gnu", or "omit". -->
                <tarLongFileMode>gnu</tarLongFileMode>
                <outputDirectory>target</outputDirectory>
                <descriptors>
                    <descriptor>src/main/assembly/package.xml</descriptor>
                </descriptors>
                <!-- 子POM的属性完全覆盖父POM的 <descriptors combine.self="override"/> -->
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id><!-- ID 标识，命名随意 -->
                    <phase>package</phase> <!--绑定到package生命周期阶段上 -->
                    <goals>
                        <goal>single</goal><!-- 在 PACKAGE 生命周期阶段仅执行一次 -->
                    </goals>
                </execution>
            </executions>
        </plugin>
    ```
描述文件--><descriptors>标签对应的package.xml文件
1. <id>与<formats>标签：
    * <id>标记打包到文件明德标识符，当依赖中的appendAssemblyId为true的时候才有用；为true时，生成的文件名为：{project.artifactId}-{id}-{版本号}
    * <formats>标签是用来定义插件打包后产生的文件样式，该标签包含多个<format>标签；支持的文件样式有：zip、tar、tar.gz、tar.bz2、jar、war；
    * 示例：
    <id>assembly-${project.version}</id>
    <!--打包输出物,一个文件夹和一个tar.gz的压缩包-->
    <formats>
        <format>dir</format>
        <format>tar.gz</format>
    </formats>
2. <includeBaseDirectory>false</includeBaseDirectory>标签，为true时会把文件进一步打包到依赖的<finalName>标签中设置的文件下；
3. <fileSets>/<fileSet>标签,用来设置工程中，哪些文件如何打包，打包到什么路径下；
    * directory：源目录的路径。
    * includes/excludes：设定包含或排除哪些文件，支持通配符。
    * fileMode：指定该目录下的文件属性，采用Unix八进制描述法，默认值是0644。
    * outputDirectory：生成目录的路径。
4. <dependencySets>/<dependencySet>标签，设置功能运行时所需的依赖文件打包的位置；
<dependencySets>
    <dependencySet>
        <!--是否把本项目添加到依赖文件夹下？-->
        <useProjectArtifact>false</useProjectArtifact>
        <!--输出路径-->
        <outputDirectory>./lib</outputDirectory>
        <!--符合什么作用范围的依赖，一般都用runtime，其他没意义-->
        <scope>runtime</scope>
    </dependencySet>
</dependencySets>


    


### 参考
1. https://www.jianshu.com/p/d44f713b1ec9  Maven maven-jar-plugin
2. https://segmentfault.com/a/1190000016237395?utm_source=tag-newest maven--插件篇（assembly插件）
3. https://www.jianshu.com/p/e581fff1cf87 使用maven-assembly-plugin插件自定义项目打包
