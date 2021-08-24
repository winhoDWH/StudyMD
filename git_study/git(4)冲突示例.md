示例：
1. 在一个仓库A中编写了1.txt文件中内容为：
    ![仓库A的修改](img/firstChange.png)
2. 然后上传到远程仓库中
    ![上传](img/firstCommit.png)
3. 修改另一个仓库B中的1.txt文件内容为（在没有同步A仓库的修改下）：
    ![仓库B的修改](img/SecondChange.png)
4. commit到仓库B都是可以的，但是在push到远程仓库时就会报错：
    ![push失败](img/push_error.png)
    >原因是与远程仓库有同一个文件修改记录不一致（本质上是日志上的指针不一致）；
5. 这时候需要pull远程仓库中的文件，因为pull包含合并merge操作，所以会报错，原因是拉取到本地仓库的文件（即与远程仓库一致firstChange文件内容）与工作区的文件（SecondChange的文件内容）不相同，不能直接合并；
    ![拉取](img/pull_error.png)
6. 根据提示进行resole（解决冲突）：
    ![解决冲突](img/resole.png)
    >1区域是远程仓库上的文件内容；2区域是本地工作区的文件内容；3区域是为解决冲突你编写的的文件内容；
    ![resole结果](img/resole_result.png)
7. 修改完成后，可进行正常的commit与push了，这样为解决冲突的全过程； 