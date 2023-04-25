## httpHesponse

- 负责传递给浏览器响应的一些消息

- 下载文件

  - （1）要获取下载文件的 路径

  - （2）下载的文件名是什么？

  - （3）设置想办法让浏览器能支持下载我们需要的东西

  - （4）获取下载文件的输入流

  - （5）创建缓冲区

  - （6）获取OutputStream对象

  - （7）将FileOutputStream流写入到buffer缓冲区

  - （8）将OutputStream将缓冲区中的数据输出到客户端

    ```java
    public class FileServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            //获取文件路径
            String filePath = this.getServletContext().getRealPath("/1.png");
            System.out.println("下载文件路径：" + filePath);
    
            //获取下载文件名
            //这样可以方便获取到最后文件名
            String fileName = filePath.substring(filePath.lastIndexOf("\\" + 1));
    
            //设置浏览器接收文件参数
            //注意转中文
            resp.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
    
            //将文件获取到IO流中 获取inputStream
            FileInputStream fileInputStream = new FileInputStream(filePath);
    
            //创建缓冲区
            byte[] buffer = new byte[1024];
    
            //获取outputStream
            ServletOutputStream out = resp.getOutputStream();
    
            //将文件IO流写入到缓冲区中
            while (fileInputStream.read(buffer) > 0) {
                out.write(buffer);
            }
    
            //关闭流
            fileInputStream.close();
            out.close();
        }
    }
    ```

- 展示图片

  ``` java
  public class ShowImageServlet extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //让浏览器每3秒刷新一次
          resp.setHeader("refresh", "3");
  
          //在内存中创建图片
          BufferedImage image = new BufferedImage(80, 20, BufferedImage.TYPE_INT_RGB);
  
          //获取图片的绘画工具-笔
          Graphics2D g = (Graphics2D) image.getGraphics();
  
          //设置背景色
          //1.先设置画笔颜色
          g.setColor(Color.blue);
          //2.填充
          g.fillRect(0, 0, 80, 20);
  
          //设置数字
          //1.先设置画笔颜色
          g.setColor(Color.black);
          //2.设置字体
          g.setFont(new Font(null, Font.BOLD, 20));
          g.drawString(getRandomStr(), 0, 20);
  
          //告诉浏览器用图片打开返回
          resp.setContentType("image/jpeg");
          //消除网站的缓存
          resp.setDateHeader("expires", -1);
          resp.setHeader("Cache-Control", "no-cache");
          resp.setHeader("Pragma", "no-cache");
  
          ImageIO.write(image, "jpg", resp.getOutputStream());
      }
  
      /**
       * 获取一个7位的随机数字字符串
       * @return
       */
      private static String getRandomStr(){
          Random random = new Random();
          //获取一个0到7位数的随机数
          String num = random.nextInt(9999999) + "";
  
          //补全随机数可能缺失的位数
          StringBuffer buffer = new StringBuffer();
          for (int i = 0; i < 7 - num.length(); i++){
              //不足7位，左边补0
              buffer.append("0");
          }
  
          return buffer + num;
      }
  }
  ```

- 重定向

  - 状态码：302
  - 使用代码：

  ``` java
  void sendRedirect(String var1) throws IOException;
  ```

- 重定向和转发的区别

  - 相同点：都是改变访问的页面，都发生跳转
  - 不同点：
    - 转发是服务器处理，url不会发生变化；
    - 重定向是浏览器处理，url会发生变化。

## HttpServletRequest

