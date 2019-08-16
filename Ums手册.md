### 注意:

我采用的是本地的数据库,用户名duanpeng;密码123456 database:ums Table:userinfo

要在本地运行的话,要先检查数据库是否启动了,然后执行ums.sql创建数据库和相应的表格,数据.

数据库里面仅仅有一条数据

username:huhui,password:123456,nikname=hubaobao

### 启动过程:

1. 启动tcpserver
2. 启动httpserver
3. 浏览器打开localhost:9090页面
4. 输入username和password,提交
5. 从数据库查询成功后,页面显示{"success":true,"msg":"found user"}

### 后续任务:

1. redis的Auth模块
   * 首次登录,调用tcpserver服务,应该返回全部user信息(此处还需定义统一的消息格式{"success":true,"msg":"","data":""},data存放user信息),生成token存入redis(key:token,value:user的json字符串)
   * 已经登录,查看redis中是否有token,有的话跳转至登录上传文件界面
2. 上传文件模块
   * 用一个FTP服务存放文件
   * 为防止文件重名,应为文件生成唯一UUID
   * 文件地址存入数据库,建profile表格,绑定用户文件的记录(id,userid,文件地址)
3. 数据库连接池
4. 为数据库生成10,000,000条数据信息