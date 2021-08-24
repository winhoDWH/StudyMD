#### 创建一个idea的maven项目
##### 忽视项目中的idea的配置文件：.dea文件与.iml文件
1. 首先在Idea中的setting设置中，搜索File Types
2. 选择图中选项，并在下方添加*.idea和*.iml两项配置说明。
3. 如果库中已有对应的.idea与.iml文件，则使用命令
    >$ git rm --cached -r *.idea(或者 *.iml)
4. 添加.gitignore文件，并在文件中加上
    >**.idea和 **.iml
