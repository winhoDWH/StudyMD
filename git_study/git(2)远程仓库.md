##GitHub
1. 创建一个GitHub账号后，获取本机的git仓库SSH KEY；
2. 获取SSH KEY：在git bash here中输入
    ```
    $ ssh-keygen -t rsa -C "youremail@example.com"
    ```
    然后不断回车确认，不需要输入key和password；
3. 在创建后的信息中可以看到ssh key生成的文件目录.ssh文件，打开后看到有两个文件，id_rsa文件是私钥，id_rsa.pub是公钥；
4. 在GitHub的setting中添加**公钥**就可以建立起所使用的电脑和GitHub之间的传输联系了；
5. GitHub使用ssh key是为了唯一识别提交的设备；GIT支持SSH协议；

##将文件上传到远程仓库GitHub中
1. 在GitHub中创建一个新的仓库；
2. 在本地仓库处，执行如下代码进行关联
    ```
    $ git remote add <远程库名> git@github.com:<github账号用户名>/<GitHub仓库名>.git
    ```
3. 关联之后可以使用**git push -u <远程库名> <分支名>**将本地库所有内容推送到远程库；加上-u参数会在本地的内容推送到远程库后会进行关联，简化之后的推送或者拉取；
4. 本地库提交（commit）之后可以使用**git push <远程库名> <分支名>**将数据推送到远程库中；
5. **git remote -v**可以查看git连接访问远程库的方式：
    ```
    // HTTPS方式
    https://github.com/xxxx/StudyEveryDay.git
    // SSH方式
    git@github.com:xxxx/StudyEveryDay.git
    ```
6. 第一次使用github连接的时候会有一个警告，直接输入yes即可；
7. **远程仓库默认名称为origin**;

##克隆
1. 先在GitHub上创建一个仓库，然后在本地使用**git clone git@....(与remote后面是一致的)**克隆在本地仓库；
