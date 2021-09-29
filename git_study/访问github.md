## 加快电脑上访问github以及拉取github代码的速度

1. 访问 https://fastly.net.ipaddress.com/ 网站，查询github.global.ssl.Fastly.net 和github.com的映射网站地址

2. 修改 C:\Windows\System32\drivers\etc下的host文件

   ``` 
   # 该地址添加后加快拉取代码速度
   151.101.113.194 github.global.ssl.fastly.net
   # 该地址添加后加快网站访问速度
   192.30.253.112 github.com
   ```

   

