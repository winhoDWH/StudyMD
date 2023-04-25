## 一、 Maven仓库的概念分类

1. maven仓库分为本地仓库与远程仓库两大类；

2. 远程仓库分为：私服仓库、网上别的远程仓库、中央仓库；

3. 本地仓库：每个用户自己电脑本地会有一个maven仓库，可以在`setting.xml`文件中配置对应的本地仓库地址；

4. 远程仓库：即除自己电脑本地以外，联网请求的maven仓库；

5. 私服仓库：远程仓库中特殊的一种，局域网自己配置的仓库，在`setting.xml`文件中使用`profile`标签配置；

6. 中央仓库：maven必须至少有一个可用的远程仓库，所以这是maven自己固定配置好的远程仓库，配置在:

   ```
   1. \apache-maven-3.0.4\lib 下的 maven-model-builder-3.0.4.jar 中的 org/apache/maven/model/pom-4.0.0.xml中，其库ID为central
   ```

7. maven镜像：`mirror`

   * 镜像原本含意：一个磁盘上的数据在另一个磁盘上完全相同的副本；

   * 在maven中配置镜像的原因是：更快更高效的获取某远程仓库中的依赖包；

   * 注意：镜像在`setting.xml中配置`，使用`<mirrorOf>`标签对应某个或者某些远程仓库，其含义为：对这些远程仓库进行的请求访问转向该镜像；

     ```
     <mirrorOf>标签配置：
     external:* = 不在本地仓库的文件才从该镜像获取
     repo,repo1 = 远程仓库 repo 和 repo1 从该镜像获取
     *,!repo1 =  所有远程仓库都从该镜像获取，除 repo1 远程仓库以外
     * = 所用远程仓库都从该镜像获取
     ```

     所以如果某一个镜像配置`central`，该maven的中心仓库就修改为对应镜像

     总结：由于镜像配置一定要匹配某一个或某些远程仓库，所以镜像实际上可以等同于一个远程仓库。

## 二、 maven仓库获取选择优先级

1. 总结：

   * 在本地仓库寻找，找不到下一步
   * 在私服仓库中寻找，找不到下一步
   * 在其他远程仓库（镜像）中寻找，找不到下一步
   * 在中央仓库中寻找，如果没有则终止寻找

2. 通过maven打印的日志可以验证上述步骤（以下节取博客）：

   ```
   仓库配置：
   私服：172.16.xxx.xxx
   远程仓库：dev.xxx.wiki 
   mirror：为本地 localhost 仓库
   中央仓库为 maven.aliyun.com 
   ```

   `setting.xml`文件

   ```xml
   <mirrors> 
     <mirror> 
       <id>localhost</id>  
       <name>Public Repositories</name>  
       <mirrorOf>foo</mirrorOf>   <!--拦截 pom 文件配置的 repository-->
       <url>http://localhost:8081/repository/maven-public/</url> 
     </mirror>  
     
     <mirror> 
       <id>localhost2</id>  
       <name>Public Repositories</name>  
       <mirrorOf>foo2</mirrorOf>   <!--配置一个拦截 foo2 的远程仓库的镜像-->
       <url>http://localhost:8081/repository/maven-snapshots/</url> 
     </mirror>  
     
     <mirror> 
       <id>alimaven</id>  
       <mirrorOf>central</mirrorOf>  <!--覆盖 Maven 默认的配置的中央仓库-->
       <name>aliyun maven</name>  
       <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
     </mirror> 
    
   </mirrors>
   <!--配置私服-->
   <profiles> 
     <profile> 
       <id>nexus</id>  
       <repositories> 
         <repository> 
           <id>public</id>  
           <name>Public Repositories</name>  
           <url>http://172.16.xxx.xxx:8081/nexus/content/groups/public/</url> 
         </repository> 
       </repositories>  
       <pluginRepositories> 
         <pluginRepository> 
           <id>public</id>  
           <name>Public Repositories</name>  
           <url>http://172.16.xxx.xxx:8081/nexus/content/groups/public</url> 
         </pluginRepository> 
       </pluginRepositories> 
     </profile> 
   </profiles>
   <activeProfiles>
       <activeProfile>nexus</activeProfile>
   </activeProfiles>
   ```

   项目`pom.xml`文件

   ```xml
   <dependencies>
       <!--xxx-cif-api 存在 172.16.xxx.xxx 仓库-->
       <dependency>
           <groupId>com.xxx.cif</groupId>
           <artifactId>xxx-cif-api</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
       <!--Chapter1 存在 localhost 仓库-->
       <dependency>
           <groupId>com.cjf</groupId>
           <artifactId>Chapter1</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
   </dependencies>
   <!--配置远程仓库-->
   <repositories>
       <repository>
           <id>foo</id>
           <name>Public Repositories</name>
           <url>http://dev.xxx.wiki:8081/nexus/content/groups/public/</url>
       </repository>
   </repositories>
   ```

   maven拉取包的日志：

   ```
   ······· 省略部分日志信息
   [DEBUG] Using local repository at E:\OperSource
   [DEBUG] Using manager EnhancedLocalRepositoryManager with priority 10.0 for E:\OperSource
   [INFO] Scanning for projects...
   // 从这里可以看出我们配置的镜像替代了我们在 pom 配置的远程仓库
   [DEBUG] Using mirror localhost (http://localhost:8081/repository/maven-public/) for foo (http://dev.xxx.wiki:8081/nexus/content/groups/public/).
   替代了默认的中央仓库
   [DEBUG] Using mirror alimaven (http://maven.aliyun.com/nexus/content/groups/public/) for central (https://repo.maven.apache.org/maven2).
   // 从这里可以看出 Maven 使用哪些 dependencies 和 plugins 的地址，我们可以看出优先级最高的是 172.16.xxx.xxx,然后就是 localhost 最后才是 maven.aliyun.com
   // 注意：alimaven (http://maven.aliyun.com/nexus/content/groups/public/, default, releases) 从这里可以看出中央仓库只能获取 releases 包，所有的 snapshots 包都不从中央仓库获取。（可以看前面 central 的配置信息）
   [DEBUG] === PROJECT BUILD PLAN ================================================
   [DEBUG] Project:       com.cjf:demo:1.0-SNAPSHOT
   [DEBUG] Dependencies (collect): []
   [DEBUG] Dependencies (resolve): [compile, runtime, test]
   [DEBUG] Repositories (dependencies): [public (http://172.16.xxx.xxx:8081/nexus/content/groups/public/, default, releases+snapshots), localhost (http://localhost:8081/repository/maven-public/, default, releases+snapshots), alimaven (http://maven.aliyun.com/nexus/content/groups/public/, default, releases)]
   [DEBUG] Repositories (plugins)     : [public (http://172.16.xxx.xxx:8081/nexus/content/groups/public, default, releases+snapshots), alimaven (http://maven.aliyun.com/nexus/content/groups/public/, default, releases)]
   [DEBUG] =======================================================================
   // 寻找本地是否有 maven-metadata.xml 配置文件 ，从这里可以看出寻找不到（后面会详细讲该文件作用）
   [DEBUG] Could not find metadata com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml in local (E:\OperSource)
   // 由于寻找不到 Maven 只能从我们配置的远程仓库寻找，由于 Maven 也不知道那个仓库才有，所以同时寻找两个仓库
   [DEBUG] Using transporter WagonTransporter with priority -1.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   [DEBUG] Using transporter WagonTransporter with priority -1.0 for http://localhost:8081/repository/maven-public/
   [DEBUG] Using connector BasicRepositoryConnector with priority 0.0 for http://localhost:8081/repository/maven-public/
   [DEBUG] Using connector BasicRepositoryConnector with priority 0.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   Downloading: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/maven-metadata.xml
   Downloading: http://localhost:8081/repository/maven-public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/maven-metadata.xml
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\resolver-status.properties
   // 从这里可以看出在 172.16.xxx.xxx 找到  xxx-cif-api 的 maven-metadata.xml 文件并下载下来
   Downloaded: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/maven-metadata.xml (781 B at 7.0 KB/sec)
   // 追踪文件，resolver-status.properties 配置了 jar 包下载地址和时间
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\resolver-status.properties
   [DEBUG] Could not find metadata com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml in localhost (http://localhost:8081/repository/maven-public/)
   // 在 localhost 远程仓库寻找不到 xxx-cif-api 的 maven-metadata.xml
   [DEBUG] Could not find metadata com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml in local (E:\OperSource)
   // 跳过的远程请求 
   [DEBUG] Skipped remote request for com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml, already updated during this session.
   [DEBUG] Skipped remote request for com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml, already updated during this session.
   // 默认以后获取 xxx-cif-api 的时候将不在从 localhost 寻找了，除非强制获取才会再次从 localhost 寻找这个包
   [DEBUG] Failure to find com.xxx.cif:xxx-cif-api:0.0.1-SNAPSHOT/maven-metadata.xml in http://localhost:8081/repository/maven-public/ was cached in the local repository, resolution will not be reattempted until the update interval of localhost has elapsed or updates are forced
   // 将 172.16.xxx.xxx 优先级升为 0 ，并下载 xxx-cif-api 的 pom 文件
   [DEBUG] Using transporter WagonTransporter with priority -1.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   [DEBUG] Using connector BasicRepositoryConnector with priority 0.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   Downloading: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/xxx-cif-api-0.0.1-20170515.040917-89.pom
   Downloaded: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/xxx-cif-api-0.0.1-20170515.040917-89.pom (930 B at 82.6 KB/sec)
   // _remote.repositories 记录的以后使用那个远程仓库获取 （ps:这个文件作用我要不是很清楚作用，以上观点是自己推测出来的。）
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\_remote.repositories
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\xxx-cif-api-0.0.1-20170515.040917-89.pom.lastUpdated
   // 后面获取 Chapter1 包的流程跟 com.xxx.cif 是一样的，不过最后是在 localhost 寻找到而已，所以这分日志就不贴出来了。
   // 最后在下载包的时候，都到对应的仓库下载
   [DEBUG] Using transporter WagonTransporter with priority -1.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   [DEBUG] Using connector BasicRepositoryConnector with priority 0.0 for http://172.16.xxx.xxx:8081/nexus/content/groups/public/
   Downloading: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/xxx-cif-api-0.0.1-20170515.040917-89.jar
   Downloading: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/util/xxx-util/0.0.1-SNAPSHOT/xxx-util-0.0.1-20170514.091041-31.jar
   Downloaded: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/util/xxx-util/0.0.1-SNAPSHOT/xxx-util-0.0.1-20170514.091041-31.jar (26 KB at 324.2 KB/sec)
   Downloaded: http://172.16.xxx.xxx:8081/nexus/content/groups/public/com/xxx/cif/xxx-cif-api/0.0.1-SNAPSHOT/xxx-cif-api-0.0.1-20170515.040917-89.jar (68 KB at 756.6 KB/sec)
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\_remote.repositories
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\cif\xxx-cif-api\0.0.1-SNAPSHOT\xxx-cif-api-0.0.1-20170515.040917-89.jar.lastUpdated
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\util\xxx-util\0.0.1-SNAPSHOT\_remote.repositories
   [DEBUG] Writing tracking file E:\OperSource\com\xxx\util\xxx-util\0.0.1-SNAPSHOT\xxx-util-0.0.1-20170514.091041-31.jar.lastUpdated
   [DEBUG] Using transporter WagonTransporter with priority -1.0 for http://localhost:8081/repository/maven-public/
   [DEBUG] Using connector BasicRepositoryConnector with priority 0.0 for http://localhost:8081/repository/maven-public/
   Downloading: http://localhost:8081/repository/maven-public/com/cjf/Chapter1/0.0.1-SNAPSHOT/Chapter1-0.0.1-20170708.092339-1.jar
   Downloaded: http://localhost:8081/repository/maven-public/com/cjf/Chapter1/0.0.1-SNAPSHOT/Chapter1-0.0.1-20170708.092339-1.jar (8 KB at 167.0 KB/sec)
   [DEBUG] Writing tracking file E:\OperSource\com\cjf\Chapter1\0.0.1-SNAPSHOT\_remote.repositories
   [DEBUG] Writing tracking file E:\OperSource\com\cjf\Chapter1\0.0.1-SNAPSHOT\Chapter1-0.0.1-20170708.092339-1.jar.lastUpdated
   [INFO] Installing C:\Users\swipal\Desktop\abc\demo\target\demo-1.0-SNAPSHOT.jar to E:\OperSource\com\cjf\demo\1.0-SNAPSHOT\demo-1.0-SNAPSHOT.jar
   [DEBUG] Writing tracking file E:\OperSource\com\cjf\demo\1.0-SNAPSHOT\_remote.repositories
   [INFO] Installing C:\Users\swipal\Desktop\abc\demo\pom.xml to E:\OperSource\com\cjf\demo\1.0-SNAPSHOT\demo-1.0-SNAPSHOT.pom
   [DEBUG] Writing tracking file E:\OperSource\com\cjf\demo\1.0-SNAPSHOT\_remote.repositories
   [DEBUG] Installing com.cjf:demo:1.0-SNAPSHOT/maven-metadata.xml to E:\OperSource\com\cjf\demo\1.0-SNAPSHOT\maven-metadata-local.xml
   [DEBUG] Installing com.cjf:demo/maven-metadata.xml to E:\OperSource\com\cjf\demo\maven-metadata-local.xml
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   [INFO] Total time: 10.549 s
   [INFO] Finished at: 2017-07-09T18:13:20+08:00
   [INFO] Final Memory: 26M/219M
   [INFO] ------------------------------------------------------------------------
   ·······
   ```




## 参考

https://swenfang.github.io/2018/06/03/Maven-Priority/