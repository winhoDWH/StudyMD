#### 数据库连接错误2059
1. 原因是：mysql18之前版本中加密规则是mysql_native_password,而在mysql8之后,加密规则是caching_sha2_password；
2. 解决方法：修改加密规则；
```
mysql -uroot -ppassword #登录

use mysql; #选择数据库
# 远程连接请将'localhost'换成'%'

ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #更改加密方式

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; #更新用户密码为password并用mysql_native_password方式加密

FLUSH PRIVILEGES; #刷新权限
```