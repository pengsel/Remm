# 简单的Tomcat服务器

### Client

1. 建立Socket对象
2. 获取输入流对象
3. 获取输出流对象
4. 将HTTP协议的请求部分发送到服务器
5. 读取服务端的数据到控制台

### Server

1. 建立ServerSocket对象，监听8080端口
2. 等待客户端的请求，获取和客户端对应的Socket对象
3. 获取输出流对象
4. 通过输出流将HTTP协议发送到客户端
5. 释放资源

#### 静态资源

1. 定义变量WEB_ROOT，存放服务器WebContent目录的绝对路径；
2. 静态变量，存放静态页面名称；
3. Server与上面基本一致，不同地方：
   * parse函数，解析请求。
   * sendStaticResource，发送静态资源。

#### 动态资源

1. Servlet接口，以及实现类
   * init()
   * service()
   * destroy()
2. conf.properties：资源目录和相应Servlet全路径名
3. 静态类型Map，存放服务器conf.properties的配置信息，服务器启动时就读取配置文件，过程如下：
   * 创建Properties对象
   * load(new FileInputStream(路径))
   * 将配置文件中的数据读取到Map中；

4. sendDynamicResource()
   * 判断map中是否存在相应key
   * 通过反射将Java程序加载到内存
   * 执行init方法，service方法