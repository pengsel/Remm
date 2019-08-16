[TOC]

## TCPServer

TCPServer的主要作用是管理Connection，获得消息Message，并实现具体的业务逻辑。因此，TCPServer的核心模块是连接管理模块ConnManager和数据处理模块MsgHandler，两者之间以Request作为通信手段，Request是Message的载体。进一步，为方便数据传输，衍生出数据的封包拆包DataPack模块，同时，为了方便TCPServer实现多种业务逻辑，使用Router包装一种业务，采用map来存储消息ID(msgId)与Router的映射，存储在MsgHandler的Apis字段中。

### 接口

#### IServer

IServer封装了一个服务器接口，主要提供服务器的启动、停止、服务等方法。

| 函数                                    | 描述                  |
| --------------------------------------- | --------------------- |
| Start()                                 | 启动服务器            |
| Stop()                                  | 停止服务器            |
| Serve()                                 | 开启服务              |
| AddRouter(msgId uint32, router IRouter) | 注册一个业务          |
| GetConnMgr() IConnManager               | 获取连接管理的Handler |

#### IConnection

IConnection接口定义了连接接口,主要实现了连接启动、停止功能，通过连接可以发送消息。

| 函数                                         | 描述                                               |
| -------------------------------------------- | -------------------------------------------------- |
| Start()                                      | 启动连接                                           |
| Stop()                                       | 停止连接                                           |
| GetTCPConnection() *net.TCPConn              | 获取原始的socket TCPConn                           |
| GetConnID() uint32                           | 获取当前连接ID                                     |
| RemoteAddr() net.Addr                        | 获取远程客户端地址信息                             |
| SendMsg(msgId uint32, data []byte) error     | 直接将Message数据发送数据给远程的TCP客户端(无缓冲) |
| SendBuffMsg(msgId uint32, data []byte) error | 直接将Message数据发送给远程的TCP客户端(有缓冲)     |
| SetProperty(key string, value interface{})   | 设置链接属性                                       |
| GetProperty(key string)(interface{}, error)  | 获取链接属性                                       |
| RemoveProperty(key string)                   | 移除链接属性                                       |

#### IConnManager

IConnManager接口用来管理TCPServer的连接，主要提供了连接的添加和删除功能，用来管理连接池。

| 函数                                    | 描述               |
| --------------------------------------- | ------------------ |
| Add(conn IConnection)                   | 添加链接           |
| Remove(conn IConnection)                | 删除连接           |
| Get(connID uint32) (IConnection, error) | 利用ConnID获取链接 |
| Len() int                               | 获取当前连接       |
| ClearConn()                             | 删除并停止所有链接 |

#### IMessage

IMessage定义了一个消息结构接口，主要用来获取MsgId和消息内容。

| 函数                | 描述               |
| ------------------- | ------------------ |
| GetDataLen() uint32 | 获取消息数据段长度 |
| GetMsgId() uint32   | 获取消息ID         |
| GetData() []byte    | 获取消息内容       |
| SetMsgId(uint32)    | 设置消息ID         |
| SetData([]byte)     | 设置消息内容       |
| SetDataLen(uint32)  | 设置消息数据段长度 |

#### IMsgHandler

IMsgHandler主要是对消息处理的管理，主要管理将消息发送给Worker工作池的任务队列中。

| 函数                                    | 描述                                 |
| --------------------------------------- | ------------------------------------ |
| DoMsgHandler(request IRequest)          | 处理消息                             |
| AddRouter(msgId uint32, router IRouter) | 为消息添加具体的处理逻辑             |
| StartWorkerPool()                       | 启动工作池                           |
| SendMsgToTaskQueue(request IRequest)    | 将消息交给TaskQueue,由worker进行处理 |

#### IRequest

IRequest封装了一个请求，用来获取请求内容和响应的请求连接，将用户的请求和连接绑定在一起。

| 函数                        | 描述               |
| --------------------------- | ------------------ |
| GetConnection() IConnection | 获取请求连接信息   |
| GetData() []byte            | 获取请求消息的数据 |
| GetMsgID() uint32           | 获取请求的消息ID   |

#### IDataPack

IDataPack是处理封包拆包数据传输的接口，主要提供封包拆包两个方法，解决了TCP粘包问题。

| 函数                              | 描述             |
| --------------------------------- | ---------------- |
| GetHeadLen() uint32               | 获取包头长度方法 |
| Pack(msg IMessage)([]byte, error) | 封包方法         |
| Unpack([]byte)(IMessage, error)   | 拆包方法         |

#### IRouter

IRouter是具体的处理业务逻辑的接口。

| 函数                         | 描述     |
| ---------------------------- | -------- |
| PreHandle(request IRequest)  | 业务之前 |
| Handle(request IRequest)     | 业务     |
| PostHandle(request IRequest) | 业务之后 |



### 实现

#### Server

Server定义了服务器的名称，IP地址和端口，链接管理和消息管理等等。

| 字段                         | 描述            |
| ---------------------------- | --------------- |
| Name string                  | 名称            |
| IPVersion string             | IP地址版本      |
| IP string                    | IP地址          |
| Port int                     | 端口号          |
| msgHandler ziface.IMsgHandle | 消息管理Handler |
| ConnMgr ziface.IConnManager  | 连接管理        |

#### MsgHandler

MsgHandler里面主要存储了消息ID与对应业务函数IRouter的映射，通过消息管理，可以将ID与对应的业务处理函数相绑定。

| 字段                                     | 描述                   |
| ---------------------------------------- | ---------------------- |
| Apis           map[uint32]ziface.IRouter | 消息id和处理函数的映射 |
| WorkerPoolSize uint32                    | 业务工作Worker的数量   |
| TaskQueue      []chan ziface.IRequest    | Worker的任务队列       |

#### ConnManager

ConnManager是一个连接池的管理模块，主要管理连接信息。里面包含了连接信息，和读写锁，方便对连接进行获取和归还。

| 字段                                      | 描述                   |
| ----------------------------------------- | ---------------------- |
| connections map[uint32]ziface.IConnection | 管理连接信息           |
| connLock    sync.RWMutex                  | 获取和归还连接的读写锁 |

#### Connection

Connection主要封装了连接的基本信息，并实现了读写分离，通过chan来实现读写之间的通信。

| 字段                            | 描述                 |
| ------------------------------- | -------------------- |
| TcpServer ziface.IServer        | 属于哪个Server       |
| Conn *net.TCPConn               | Socket套接字         |
| ConnID uint32                   | 当前连接ID           |
| isClosed bool                   | 关闭状态             |
| MsgHandler ziface.IMsgHandle    | 消息处理模块         |
| ExitBuffChan chan bool          | 告知该连接退出       |
| msgChan chan []byte             | 读写之间的消息通信   |
| msgBuffChan chan []byte         | 有缓冲管道，读写通信 |
| property map[string]interface{} | 连接属性             |
| propertyLock sync.RWMutex       | 修改连接属性的读写锁 |

#### Request

Request封装了与客户端建立好的链接，以及客户端请求的数据。

| 字段                    | 描述             |
| ----------------------- | ---------------- |
| conn ziface.IConnection | 与客户端的链接   |
| msg ziface.IMessage     | 客户端的请求数据 |

#### Message

Message代表一个请求实体，包括请求数据和相应处理函数(即Router))的ID。为了方便封包，还有一个DateLen字段。

| 字段           | 描述       |
| -------------- | ---------- |
| DataLen uint32 | 消息的长度 |
| Id      uint32 | 消息的ID   |
| Data    []byte | 消息的内容 |

#### BaseRouter

BaseRouter定义了一个模板，可以选取其中的Handler实现，不必实现所有的函数。

#### DataPack

DataPack实现了一种封包拆包的方法，按照(msgID(4字节)+DataLen(4字节)+Data)的格式封包，拆包时分两次，先读取8个字节，再读取后面的数据。

## RPC

#### 基本原理

RPC模块是通过网络从远程计算机上请求服务，而不需要了解底层网络技术的协议。在ums中，RPC模块采用了固定的入参和出参，均为Message对象，Message对象包含一个msgID和Data，其中msgId为对应一个处理函数业务，Data为json字符串，当作为请求时里面包含了入参的json字符串，作为响应时包含了相应的响应json字符串。

#### 调用过程

1. TCPServer中新建多种服务，对应于多个Router，即新建了MsgHandler模块，包含msgId和处理函数的映射。
2. Router将从入参Message中提取所需要的信息，并在完成处理后将结果写入响应的json字符串，包装成一个Message对象返回给客户端。
3. 用户往TCP中写入Message，里面包含一个msgID，和相应的数据，数据为json格式，进行rpc调用后，server相应的处理函数可以从中取出所需的参数。
4. TCPServer的Router处理，结束处理后以返回结果，rpc模块接收消息后从json字符串中读取结果。

#### 处理优化

为了避免调用过程中多次获取和关闭连接造成过多的性能消耗，为rpc调用时每次从连接池获取一个TCP连接，处理完业务之后将连接归还至连接池中。

该连接池采用一个chan来实现，获取连接即从chan中得到一个连接，释放连接时将连接压入chan。



## 全局配置模块

为了方便配置整个项目，提供了一个全局配置点。采用globalobj来读取配置文件conf/ums.json，解析至GlobalObj，当TCPServer或其他模块需要配置时，直接从GlobalObj获取。