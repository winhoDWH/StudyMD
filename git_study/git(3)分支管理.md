Git与SVN等其他版本控制系统在分支管理的优势是：Git速度快，都能在1秒内完成；SVN等太慢；

##创建与合并分支
1. 原理：
    * 分支的实质是指针；master是主分支，实际上就是一个指针，HEAD也是一个指针；初始情况下，HEAD指向master，master指向当前提交点；
    * 如果我们创建了一个分支dev，则实际上创建了一个dev指针指向当前master指向的节点，HEAD也改变指向dev；
    * 后面继续对工作区文件进行修改和提交，都是在更新dev指针的指向，master不变；
    * 合并分支即为将master指向dev指向的节点，并将HEAD重新指向master；
2. 创建dev分支并切换：
    ```
    git checkout -b <分支名>
    ```
    >**git checkout**后面加上-b即为创建并切换分支；
3. **git branch <分支名>**仅仅是创建分支；**git branch**不加分支名则为查看当前有多少分支；
4. **git checkout <分支名>**直接加分支名则仅为切换分支；
5. **git merge <需要合并的分支名>**合并分支；默认是Fast-forward模式合并；
6. **git branch -d <分支名>**删除某一条分支；
7. **git switch**也是用来切换分支，使用这个更好；

##冲突
1. 场景：在一个分支A上修改了文件R的内容，然后切换成主分支master后也对同一个文件R的内容进行了不一样的修改，然后进行合并会发生冲突；**必须解决冲突才能合并**；
2. 解决冲突：将GIT合并失败的文件R再修改一次，然后进行合并就可以了；所以合并失败后的修改要修改成自己想要的样子；
3. 使用**git log --graph --pretty=oneline --abbrev-commit**命令可以看到分支图；

##git pull与git fetch的区别
1. git pull是git fetch与git merge两个指令的结合；正常流程先是将远程仓库中的文件fetch拉取到我们的**本地仓库中**，然后再**同步merge到我们的工作区中**，所以是两个指令完成一个远端仓库与本地仓库的同步；pull指令则是自动进行合并；
2. 当本地仓库与远程仓库发生冲突时（即日志中远程仓库的修改origin/master与本地的HEAD/master不一致，两个仓库的对同一个文件的修改记录不一致）：本地上传push会报错，然后就需要我们先pull或者fetch文件，使用pull的话会在merge的时候警告文件有冲突，需要我们自己去做修改
![git远程与本地操作示意图.jpg](img/git远程与本地操作示意图.jpg "git远程与本地操作示意图")

##BUG分支
1. 应用场景：在一个dev分支开发过程中，发现主分支master上文件有BUG需要修改，但现在合并dev分支会使master文件出现错误（还没有开发完），这时候就需要**隐藏dev分支的内容是我们能在保存开发进度的同时，修复以前的BUG**；
2. **git stash**命令，用于将分支内容进行隐藏，执行该命令后，git status显示工作区状态为干净的；
3. 假定需要在master分支上进行修复bug。则在master分支上另开一个分支debug1；修改完成后合并到master中；
4. 返回到dev分支上，使用**git stash list**命令可以查看之前保存的工作区记录；
5. 恢复：
    * 一是使用**git stash apply <stash代号>**，stash内容不删除，需要而外使用**git stash drop<stash代号>**删除；stash代号通过使用git stash list查看；
    * 二是使用**git stash pop**，会在恢复的同时删除内容；
6. 但是以上步骤有一个问题，就是master的bug，dev分支上也会有，所以也需要修复（问题）。解决方法：将之前debug1分支上修改后的提交复制到dev分支上；
7. **git cherry-pick <commit id>**这个命令可以复制一个commit的修改；其中commit id为debug1分支中我们提交的操作id；
8. 总结过程：隐藏开发工作的修改（stash）是我们能切换分支->修复后换回开发分支，将隐藏的修改恢复以及复制修改操作。

##删除一个还未合并的分支
使用**git branch -D <分支名>**指令；因为未合并的分支是不允许直接删除的，所以需要使用-D强制删除；

##多人协作
1. 在远程仓库有多个分支的情况下，可以选择本地分支推送到远程仓库的什么分支上，使用**git push origin(远程仓库名) <远程分支名>**;
2. 使用**git branch --set-upstream-to<本地branch-name> origin/<branch-name>**将本地分支与远程分支建立连接；
3. 根据远程分支建立对应的本地分支（**与2不相同！**）；使用**git checkout -b branch-name origin/branch-name**
4. 建立连接的目的是：**可以直接进行pull拉取，即直接使用git pull指令**；可是push上传同样还是需要标明远程分支，即使用**git push origin <远程分支名>**指令格式；

##rebase操作
使历史整理成一条直线？