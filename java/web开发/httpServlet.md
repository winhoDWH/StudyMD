#### HttpServletRequest
1. 是一个接口

#### org.apache.http
1. 当Java程序通过HTTP请求别的web服务的时候，需要通过一个工具类去组装http请求获取响应，这时候Java程序相当于一个浏览器（客户端），只发送http请求以及接收响应；
而HttpServletRequest与其不同，这是一个接口，用来统一定义web服务器或容器对Http请求进行处理后生成的HttpRequest对象，所以会带一个Servlet，因为这是服务器接收到请求之后生成的，而org.apache.http包中是不会有Servlet，因为不经过服务器。