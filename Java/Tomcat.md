## Catalina架构

连接器+容器

### 连接器

用来连接容器里面的请求，它的工作是为接受到的每一个HTTP请求构造一个Request和Response对象，然后将流程传递给容器。

### 容器

接受到Request和Response对象之后调用Servlet的service方法用于响应。包括了加载Servlet，验证用户,更新用户会话等。



### StringManager

Tomcat处理错误消息的方法是将错误消息存储子啊一个properties文件，为了避免使用一个文件而造成日志混乱，将properties文件划分到不同的包中，每个properties文件都是StringManager创建的，运行时Tomcat会有多个StringManager实例，指向多个properties文件。当查找错误消息时，会获取对应的StringManager，同一个包中使用同一个StringManager对象。

### 何时解析

在请求体和查询字符串被Servlet使用之前，连接器不会去解析它们。HttpProcessor的Parse函数会解析请求行和请求头，但不会解析请求体和查询字符串。

### 使用外观类保护Request和Response

> 外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。
>

### 注意Content-length和chunked字段

客户端Responsebody的大小，服务器有三种方式告诉你。

* . 服务器已经知道资源大小，通过content-length这个header告诉你。

```
Content-Length:1076(body的大小是1076B，你读取1076B就可以完成任务了）

Transfer-Encoding: null
```

* 服务器没法提前知道资源的大小，或者不愿意花费资源提前计算资源大小，就会把http回复报文中加一个header叫Transfer-Encoding:chunked，就是分块传输的意思。每一块都使用固定的格式，前边是块的大小，后面是数据，然后最后一块大小是0。这样客户端解析的时候就需要注意去掉一些无用的字段。

```
Content-Length:null

Transfer-Encoding:chunked (接下来的body我要一块一块的传，每一块开始是这一块的大小，等我传到大小为0的块时，就没了）
```

*  服务器不知道资源的大小，同时也不支持chunked的传输模式，那么就既没有content-length头，也没有transfer-encoding头，这种情况下必须使用短连接，以连接结束来标示数据传输结束，传输结束就能知道大小了。这时候服务器返回的header里Connection一定是close。

```
Content-Length:null

Transfer-Encoding:null

Connection:close(我不知道大小，我也用不了chunked，啥时候我关了tcp连接，就说明传输结束了）
```

### HTTP1.1的新特性

#### 持久连接 connecttion: keep-alive

返回的网页上会包含一些其他资源，服务器不会立即关闭连接，它会等待可无端请求该页面所引用的其他资源

#### 块编码 Transfer-encoding: chunked

表明字节流将会分块发送，对于每一个块，块的长度+CR/LF(回车/换行)，然后是具体内容，结束时发送一个0\r\n。

#### 状态码100 expect: 100-continiue

当客户端准备发送一个较长的请求体，但不确定服务端是否会接受时，可以加上该请求头。这样避免客户端发送了很长的请求头，而服务端不接受造成的资源浪费。服务端处理该请求时，发送响应头：

```
HTTP/1.1 100 Continue
```



Connector接口

* 连接器和Servlet容器一一对应。



HttpConnector类

* 创建服务器套接字
* 维护HttpProcessor对象池，每个HttpProcessor实例都运行在自己的线程中，会解析HTTP请求行和请求头（注意没有请求体），填充Request对象。
* 提供HTTP请求服务

HttpProcessor类

* assign()方法的异步实现





```java
/**
 * The background thread that listens for incoming TCP/IP connections and
 * hands them off to an appropriate processor.
 */
public void run() {
    // Loop until we receive a shutdown command
    while (!stopped) {
        // Accept the next incoming connection from the server socket
        Socket socket = null;
        try {
            //                if (debug >= 3)
            //                    log("run: Waiting on serverSocket.accept()");
            socket = serverSocket.accept();
            //                if (debug >= 3)
            //                    log("run: Returned from serverSocket.accept()");
            if (connectionTimeout > 0)
                socket.setSoTimeout(connectionTimeout);
            socket.setTcpNoDelay(tcpNoDelay);
        } catch (AccessControlException ace) {
            log("socket accept security exception", ace);
            continue;
        } catch (IOException e) {
            //                if (debug >= 3)
            //                    log("run: Accept returned IOException", e);
            try {
                // If reopening fails, exit
                synchronized (threadSync) {
                    if (started && !stopped)
                        log("accept error: ", e);
                    if (!stopped) {
                        //                    if (debug >= 3)
                        //                        log("run: Closing server socket");
                        serverSocket.close();
                        //                        if (debug >= 3)
                        //                            log("run: Reopening server socket");
                        serverSocket = open();
                    }
                }
                //                    if (debug >= 3)
                //                        log("run: IOException processing completed");
            } catch (IOException ioe) {
                log("socket reopen, io problem: ", ioe);
                break;
            } catch (KeyStoreException kse) {
                log("socket reopen, keystore problem: ", kse);
                break;
            } catch (NoSuchAlgorithmException nsae) {
                log("socket reopen, keystore algorithm problem: ", nsae);
                break;
            } catch (CertificateException ce) {
                log("socket reopen, certificate problem: ", ce);
                break;
            } catch (UnrecoverableKeyException uke) {
                log("socket reopen, unrecoverable key: ", uke);
                break;
            } catch (KeyManagementException kme) {
                log("socket reopen, key management problem: ", kme);
                break;
            }

            continue;
        }

        // Hand this socket off to an appropriate processor
        HttpProcessor processor = createProcessor();
        if (processor == null) {
            try {
                log(sm.getString("httpConnector.noProcessor"));
                socket.close();
            } catch (IOException e) {
                ;
            }
            continue;
        }
        //            if (debug >= 3)
        //                log("run: Assigning socket to processor " + processor);
        processor.assign(socket);

        // The processor will recycle itself when it finishes

    }
```

HttpConnector的主要方法，将TCP、IP连接扔给合适的HttpProcessor，最后调用了processor.assign()方法，这个方法是在conn这个线程上执行的，为了让它能够异步实现，即一个HttpProcessor对象可以同时处理多个请求。

它是一个同步方法，进入前会获取当前对象的锁，conn的当前线程获取到锁后，看processor是不是会wait()，直到有一个processor线程处理完毕时将其唤醒notify()，然后进入等待队列。

available一开始是false，processor线程会先启动，然后全部进入等待，当conn线程新来了一个连接请求时，它会执行processor.invoke()函数，此时conn线程将socket交给processor，并把该processor锁上等待的线程唤醒，此后processor会调用process函数处理该Socket。

conn线程会从其维护的processor池中获取一个在等待的processor，交给它任务后就继续回去监听了，它不用管任何此后任何处理。

processor如何从conn线程获取一个Socket呢？采用的就是wait()和notifyAll()方法，conn线程获取到processor的锁之后，将其赋值给processor对象，完成赋值后调用notifyAll唤醒processor线程，唤醒了之后processor线程完成具体任务。

```java
/**
 * Await a newly assigned Socket from our Connector, or <code>null</code>
 * if we are supposed to shut down.
 */
private synchronized Socket await() {

    // Wait for the Connector to provide a new Socket
    while (!available) {
        try {
            wait();
        } catch (InterruptedException e) {
        }
    }

    // Notify the Connector that we have received this Socket
    Socket socket = this.socket;
    available = false;
    notifyAll();

    if ((debug >= 1) && (socket != null))
        log("  The incoming request has been awaited");

    return (socket);

}
```

```java
/**
 * Process an incoming TCP/IP connection on the specified socket.  Any
 * exception that occurs during processing must be logged and swallowed.
 * <b>NOTE</b>:  This method is called from our Connector's thread.  We
 * must assign it to our own thread so that multiple simultaneous
 * requests can be handled.
 *
 * @param socket TCP socket to process
 */
synchronized void assign(Socket socket) {

    // Wait for the Processor to get the previous Socket
    while (available) {
        try {
            wait();
        } catch (InterruptedException e) {
        }
    }

    // Store the newly available Socket and notify our thread
    this.socket = socket;
    available = true;
    notifyAll();

    if ((debug >= 1) && (socket != null))
        log(" An incoming request is being assigned");

}
```

 四种类型的容器：Engine、Host、Context、Wrapper。

