#tag
标签是用来标记修改版本，方便查找修改的历史；

##创建标签
1. 使用**git tag <标签名>**，默认对当前分支的当前commit上打标签；
2. 使用**git tag <标签名> <commit id>**对指定commit id修改的示例进行标记；
3. 使用**git tag**可以查看有多少个标签；
4. 使用**git show <标签名>**查看对应标签信息；
5. 可以在**git tag**指令后添加**属性-a指定标签名，-m注解说明**

##操作标签
1. **git tag -d<标签名>**删除指定标签名的标签；
2. 推送标签到远程仓库：**git push origin <tagname>**;这种情况下如果需要删除，则要先在本地删除标签后，使用**git push origin :refs/tags/<tagname>**指令进行删除；