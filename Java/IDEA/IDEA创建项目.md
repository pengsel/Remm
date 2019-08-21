# IDEA创建项目

### 创建一个没有模板的maven项目

1. new->project，（不使用maven模板)，next
2. GroupId、ArtifactId
3. ProjectName

### 项目部署

1. Project：SDK设置
2. Modules：没有使用maven模板，需手动添加
   * 添加Web服务组件
   * 为Web设置资源目录，Web Resource Directory，一般设置为src/main/下面的webapp目录
   * 为Web设置描述文件的目录，Deployment Descriptor，设置在webapp目录下
3. Facts：表示当前项目的适配服务组件
4. Artifacts：当前项目发布的信息
   * 添加Web Application Exploded->From Modules
   * output root描述了当前项目的编译目录及适配服务(即对应Module)
   * 如有需要，在WEB-INF目录下添加lib包
5. 部署到Tomcat
   * 添加Tomcat Server
   * Deployment，添加项目发布信息
   * 添加Tomcat依赖

### ps：

1. Web Exploded：指项目未压缩，一次便于修改文件时立刻显现，建议开发时使用这种模式
2. WEB-INF：Java的WEB应用的安全目录。所谓安全就是客户端无法访问，只有服务端可以访问的目录。如果想在页面中直接访问其中的文件，必须通过web.xml文件对要访问的文件进行相应映射才能访问。
   * web.xml：描述了 servlet 和其他的应用组件配置及命名规则
   * classes/：站点所有用的 class 文件
   * lib/：web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如[数据库](http://lib.csdn.net/base/14)驱动jar文件。
   * src/：源码目录，按照包名结构放置各个[Java](http://lib.csdn.net/base/17)文件。
3. META-INF：meta information（元素信息），用来配置应用程序、扩展程序、类加载器和服务manifest.mf文件，在用jar打包时自动生成。
4. war文件：Web存档文件，包含Web应用程序的所有内容，减少了传输文件所需要的时间。
5. Jar文件：（扩展名为. Jar，Java Application Archive）包含[Java](http://whatis.ctocio.com.cn/searchwhatis/403/5948403.shtml)类的普通库、资源（resources）、辅助文件（auxiliary files）等。

