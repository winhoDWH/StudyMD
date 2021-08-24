### 安装与运行
1. 安装的版本为7.9；需要jdk环境，安装jdk为1.8版本；
2. 使用迅雷复制下载链接下载快，下载官网es的安装包elasticsearch-7.9.3-linux-aarch64.tar.gz；
3. 上传并解压文件到虚拟机上；
4. 使用命令` tar -zxvf elasticsearch-7.9.3-linux-aarch64.tar.gz`进行解压后，切换成非root用户，进入文件夹的bin目录下，使用`./elasticsearch -d`即可启动es；(不输入-d的话，启动会占用命令窗口)
5. 为了更好的使用，可以修改参数：
    * 修改存储目录：首先使用`mkdir -p [存储目录]`创建一个存储文件，然后赋予启动es的非root用户目录权限,指令示例如下：
    ```
    mkdir -p /data/elasticsearch
    chmod -R [用户名] /data/elasticsearch/
    ```
    * 修改配置文件elasticsearch.yml，建议修改之前备份原文件
    ```
    #指令
    [root@master-node ~]#cd /usr/local/elasticsearch/config       //进入配置文件所在路径
    [root@master-node config]# cp elasticsearch.yml elasticsearch.yml.bak //备份配置文件
    [root@master-node config]# vim elasticsearch.yml
    #修改配置文件中的信息
    cluster.name: ELK-Cluster
    node.name: master-node
    #存储文件所在路径
    path.data: /data/elasticsearch     
    #日志文件所在路径
    path.logs: /usr/local/elasticsearch/logs
    #外部访问的地址，可设置成0.0.0.0成直接访问该机器地址即可
    network.host: 0.0.0.0
    http.port: 9200
    ```
    * 修改相关内核参数
    ```
    [root@master-node config]# echo "vm.max_map_count=262144" >> /etc/sysctl.conf
    [root@master-node ~]# sysctl -p 
    [root@master-node ~]# vim /etc/security/limits.conf
    * soft nproc 65536
    * hard nproc 65536
    * soft nofile 65536
    * hard nofile 65536
    ```

### 参考
1. https://blog.51cto.com/hwg1227/2299995

### 启动es遇到的坑
#### 第一个问题：-bash: /usr/java/jdk1.8.0_60/bin/java: Permission denied
1. 问题定位：使用java -version命令验证后发现这是当前jdk没有权限运行；
2. 解决方法：在jdk的文件夹下，如进入/usr/java下，执行chmod 777 jdk1.8.0_60/bin/java指令即可；

#### 第二个问题：future versions of Elasticsearch will require Java 11; your Java version from [/usr/java/jdk1.8.0_60/jre] does not meet this requirement
在启动es时会弹出此提示；该提示的原因是：es7.9版本是默认使用jdk11的，而本机安装了jdk8，但由于向下兼容，所以不影响使用与启动，仅仅是提示；

#### 第三个问题：Exception in thread "main" java.io.IOException: Cannot run program "/usr/java/jdk1.8.0_60/jre/bin/java": error=13, Permission denied
    启动es时提示报错，与第一个报错一样，使用chmod指令赋权限即可；

#### 第四个问题：不能使用root用户启动
首先简单看一下报错
```
[2020-11-12T22:48:46,580][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [localhost.localdomain] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:174) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:127) ~[elasticsearch-cli-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:126) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-7.9.3.jar:7.9.3]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:111) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:178) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:393) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:170) ~[elasticsearch-7.9.3.jar:7.9.3]
        ... 6 more
uncaught exception in thread [main]
java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:111)
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:178)
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:393)
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:170)
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161)
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86)
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:127)
        at org.elasticsearch.cli.Command.main(Command.java:90)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:126)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92)
For complete error details, refer to the log at /home/elasticsearch-7.9.3/logs/elasticsearch.log
```
问题解决方法：
1. 新建用户：使用"adduser [用户名]"指令新建；使用"passwd [用户名]"指令，则可以给新建的用户设置密码；
2. 新建用户没有es文件权限，使用：chown -R [用户名] /home/elasticsearch-7.9.3 赋予用户es文件的权限；
3. 使用su [用户名]指令即可切换用户；并进行启动
```
#示例
adduser es
chown -R es /home/elasticsearch-7.9.3
su es
./elasticsearch -d
```

#### 第四个问题 
1. 报错信息：
```
[2020-11-12T23:06:03,988][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [localhost.localdomain] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: ElasticsearchException[Failure running machine learning native code. This could be due to running on an unsupported OS or distribution, missing OS libraries, or a problem with the temp directory. To bypass this problem by running Elasticsearch without machine learning functionality set [xpack.ml.enabled: false].]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:174) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:127) ~[elasticsearch-cli-7.9.3.jar:7.9.3]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:126) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-7.9.3.jar:7.9.3]
Caused by: org.elasticsearch.ElasticsearchException: Failure running machine learning native code. This could be due to running on an unsupported OS or distribution, missing OS libraries, or a problem with the temp directory. To bypass this problem by running Elasticsearch without machine learning functionality set [xpack.ml.enabled: false].
        at org.elasticsearch.xpack.ml.MachineLearning.createComponents(MachineLearning.java:645) ~[?:?]
        at org.elasticsearch.node.Node.lambda$new$14(Node.java:522) ~[elasticsearch-7.9.3.jar:7.9.3]
        at java.util.stream.ReferencePipeline$7$1.accept(ReferencePipeline.java:267) ~[?:1.8.0_60]
        at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1374) ~[?:1.8.0_60]
        at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481) ~[?:1.8.0_60]
        at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471) ~[?:1.8.0_60]
        at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708) ~[?:1.8.0_60]
        at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[?:1.8.0_60]
        at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499) ~[?:1.8.0_60]
        at org.elasticsearch.node.Node.<init>(Node.java:526) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.node.Node.<init>(Node.java:277) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:227) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:227) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:393) ~[elasticsearch-7.9.3.jar:7.9.3]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:170) ~[elasticsearch-7.9.3.jar:7.9.3]
        ... 6 more
```
2. 看报错是要设置什么配置信息->百度后得知，进入config文件夹下，修改elasticsearch.yml文件；添加xpack.ml.enabled: false 配置信息即可；

#### 第五个问题：
1. 报错信息：
```
ERROR: [1] bootstrap checks failed
[1]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
ERROR: Elasticsearch did not exit normally - check the logs at /home/elasticsearch-7.9.3/logs/ELK-Cluster.log
```
2. 解决方法：之前修改过elasticsearch.yml文件，加入了node.name属性，所以也要加入cluster.initial_master_nodes属性,其值对应node.name属性的值；具体为：
```
#配置信息
   xpack.ml.enabled: false
   cluster.name: ELK-Cluster
   node.name: master-node
   path.data: /data/elasticsearch
   network.host: 192.168.126.149
   http.port: 9200
   cluster.initial_master_nodes: ["master-node"]
```


