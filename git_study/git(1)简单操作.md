##安装git
1. 安装完成后，点击右键会有git bash here这个选项，表明安装成功；
2. 设置,在git bash here中输入：
    ```
    $ git config --global user.name "Your Name"
    $ git config --global user.email "email@example.com"
    ```
    >设置邮箱和用户名；使用git config命令的--global之后机器上的所有git仓库都会使用该配置；
3. git使用C来写的

##版本库、仓库、repository
1. 可以理解成一个目录，实际上就是电脑存放文件的地方，在对应的目录下使用git命令git init可将一个目录变成仓库；**变成仓库后目录中会多一个.git文件**；
2. 仓库可以跟踪所有文件的删除、修改、添加操作的记录；可以告诉你**文本文件**修改了什么东西；没法跟踪word文件，因为word是二进制格式的文件；
3. 题外：window自带的记事本在保存UTF-8编码的文件时，会在文件开头添加Oxefbbf（十六进制）的字符，这会导致很多编程软件识别时发生错误；
4. 讲一个创建在git文件目录里面的文件（如：readme.md）放入git仓库中：
    * 第一步：使用**git add**命令告诉Git,将文件放入仓库中：
    ```
    //即命令格式为：git add 文件名
    $git add readme.md
    ```
    * 第二部：使用**git commit** 命令将文件提交到仓库中：
    ```
    //git commit -m <message>
    $git commit -m "commit a file"
    ```
    >-m这个参数是代表对这次提交操作的描述；
    * 完成后会有提示，如：
    ```
    [master (root-commit) eaadf4e] commit a file
    1 file changed, 2 insertions(+)
    create mode 100644 readme.md
    ```
    >1 file changed:一个文件被修改了，2 insertions(+)：readme文件里面有两行内容；
5. **commit一次可以提交很多文件，所以使用add先将文件添加，然后一起commit；**

##查看git仓库中文件的修改
1. **git status**命令查看仓库状态，如：
    ```
    $ git status
    On branch master
    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   readme.txt

    no changes added to commit (use "git add" and/or "git commit -a")
    ```
    >modified这一行说明有哪些文件被修改；
2. **git diff <文件名>**命令可以查看文件**在提交前**被修改了什么地方，如果提交了就不能查看了；

##版本回滚
1. **git log**查看文件提交从最近到最远的日志；加上--pretty=oneline参数，可以减少日志输出的信息；
2. 每一次commit会有一个commit id，是一个SHA1计算出来的一个非常大的数字，用十六进制表示。
3. git中使用**HEAD**来表示当前版本，上一个版本为：**HEAD^**，用尖括号^表示上一个；上上个版本为:HEAD^^;如果版本间隔较多，则使用**HEAD~N(N为数字)**表示上N个版本；
4. **git reset**命令可以回滚版本;整个回滚命令为：
    ```
    //回滚到上一个版本
    $git reset --hard HEAD^
    ```
    >--hard参数
6. 也可以使用commit id来表示滚回到哪个版本，commit id可以不用写全，git会自动搜索；
7. git回退版本实际上试讲HEAD指针调整到另一个版本中并更新文件；
8. **git reflog**可以显示每一次操作命令；与log不同，**log显示的是每一次commit提交的记录，reflog是所有操作记录**；

##git中的分区
1. 工作区：即我们存放文件的目录；
2. 版本库：.git文件里面就是版本库；版本库还分为stage暂存区和git自动创建的第一个分区master以及一个指向master的指针HEAD;
3. 工作区、暂存区以及master之间的关系：**我们创建文件在工作区中，如创建一个readme.md文件，然后通过add命令将文件添加到暂存区中，再使用commit将暂存区的文件提交到master当前分支上**；

##git跟踪的是修改而不是文件
这句话的理解是：git通过add讲一个文件的修改（设这个修改为A）添加到暂存库中，如果在这时候再次修改了文件（设这个修改为B），然后不add修改B直接commit，这时候分支中只提交了修改A；所以git的操作是围绕一次修改为单位的；

##撤销修改
1. **cat <文件>**可以查看文本文件内容；
2. **git checkout --<文件（如：readme.md）>**，可以将**工作区**的文件状态改为上一次commit或者add即在**暂存区（不为空时）或者当前分支**的文件状态；
3. **git reset HEAD <file>**可以将暂存区的文件修改撤销掉（unstage）；
4. 使用2是将还没提交的工作区的修改撤销掉；3可以和2组合使用，将暂存区的修改撤销掉并撤销工作区的修改（2的操作）；

##删除文件
1. **git rm <file>**命令是在**工作区删除一个文件后（即自己删除了一个文件）**使用，和add是一样的，将删除操作的修改存放在暂存区，还是需要**commit**将这个修改告诉版本库；
2. 另一个情况，版本库中有你之前误删的文件，你要恢复工作区的文件，使用**git checkout --<file>**；
