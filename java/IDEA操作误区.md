#执行类配置Configuration的working directory
>working directory即工作目录，也是classpath;
>默认classpath包括：1.jdk的bin目录；2.引用的其他jar包路径；3.工程根路径。
>所以在不修改working directory的情况下，使用文件流查找文件时，文件应放在项目根目录下而不是class文件或者java文件路径中。
>实现了源代码文件和资源文件的分开管理。