
title: 使用mongoose构建web服务器
date: 2016-08-20 12:00:00 +0800
update: 2016-08-20 12:00:00  +0800
author: me
#cover: -/images/default.png
tags:
    - C++
    - WebServer
    - WebSocket
    - mongoose

---

最近在使用[mongoose](https://github.com/cesanta/mongoose)包装RESTful服务接口，内部系统是公司内部服务框架，通信协议都是二进制，每个服务的输入和输出都能自动绑定到JSON格式，和外部系统交互统一通过RESTful接口进行，在使用mongoose进行服务端和客户端基础库的包装时也发现这个库的一些不足。

<!--more-->

#### http请求例子

通常的http请求头如下:

```
GET / HTTP1.1\r\n
Host: 127.0.0.1:8080\r\n
\r\n
```
简单的http回复头:

```
HTTP1.1 200 OK\r\n
\r\n
```

mongoose是事件驱动一个库，使用mg_mgr_poll进行事件检测

核心事件如下：

```
#define MG_EV_POLL 0    /* Sent to each connection on each mg_mgr_poll() call */
#define MG_EV_ACCEPT 1  /* New connection accepted. union socket_address * */
#define MG_EV_CONNECT 2 /* connect() succeeded or failed. int *  */
#define MG_EV_RECV 3    /* Data has benn received. int *num_bytes */
#define MG_EV_SEND 4    /* Data has been written to a socket. int *num_bytes */
#define MG_EV_CLOSE 5   /* Connection is closed. NULL */
#define MG_EV_TIMER 6   /* now >= conn->ev_timer_time. double * */
```

对注释进行一下翻译

```
#define MG_EV_POLL 0    /* 调用mg_mg_poll时向各个连接触发该事件（无参数） */
#define MG_EV_ACCEPT 1  /* 服务器accept到新的连接产生该事件（参数为socket地址） * */
#define MG_EV_CONNECT 2 /* 客户端连接服务器返回该事件（参数表示成功或者失败） *  */
#define MG_EV_RECV 3    /* 收到数据（参数为已接收数据大小） */
#define MG_EV_SEND 4    /* 数据已发送（参数为已发送数据大小） */
#define MG_EV_CLOSE 5   /* 连接关闭触发该事件 */
#define MG_EV_TIMER 6   /* 定时器事件* */
```

通常不会直接使用MG_EV_SEND，但是在调用msg_send函数（其它函数比如mg_printf都会调用mg_send进行数据发送）只是将数据添加到缓冲中，并不会真正发送，所以对大文件进行传输时不能一次性将所有数据塞入缓冲区，这会导致内存暴涨，合理做法是按chunk发送，发送完一个chunk后在MG_EV_SEND事件中进行下一个chunk的发送。

对于一次普通的http请求，客户端使用mg_connet或者相关函数建立http连接后发送http头和内容，服务端收到MG_EV_REQUET，服务端使用mg_send或者相关函数发送回复，客户端收到MG_EV_REPLY事件，如果客户端是按chunked编码（tranfer-encoding: chunked）,那么服务端首先收到的是MG_EV_CHUNK（多个）事件，最后收到MG_EV_REQUET事件

> MG_EV_REQUET  
> MG_EV_REPLY  
> MG_EV_CHUNK

基本步骤描述如下：

1. 客户端发送请求
2. 服务端收到MG_EV_REQUET
3. 服务端进行回复
4. 客户端收到MG_EV_REPLY

#### 使用chunked方式编码变长数据

http头：Transfer-Encoding: chunked

如果是按chunk方式发送数据则描述如下:

1. 客户端发送请求头（包含tranfer-encoding: chunked）
2. 客户端使用mg_send_http_chunk或相关函数发送数据
3. 服务端收到MG_EV_CHUNK
4. 如果客户端还有chunk数据，回到2
5. 服务端收到MG_EV_REQUET
6. 服务端进行回复
7. 客户端收到MG_EV_REPLY

使用chunk方式发送数据，不用指定Content-Length头，[这里](http://www.jmarshall.com/easy/http/#http1.1c2)对chunk编码进行了详细说明

>Chunked Transfer-Encoding  
> 
>If a server wants to start sending a response before knowing its total length (like with long script output), it might use the simple chunked transfer-encoding, which breaks the complete response into smaller chunks and sends them in series. You can identify such a response because it contains the "Transfer-Encoding: chunked" header. All HTTP 1.1 clients must be able to receive chunked messages.
>  
> A chunked message body contains a series of chunks, followed by a line with "0" (zero), followed by optional footers (just like headers), and a blank line. Each chunk consists of two parts:
>  
> a line with the size of the chunk data, in hex, possibly followed by a semicolon and extra parameters you can ignore (none are currently standard), and ending with CRLF.
the data itself, followed by CRLF.
So a chunked response might look like the following:
> 
> HTTP/1.1 200 OK  
> Date: Fri, 31 Dec 1999 23:59:59 GMT  
> Content-Type: text/plain  
> Transfer-Encoding: chunked  
> 
> 1a; ignore-stuff-here
> abcdefghijklmnopqrstuvwxyz  
> 10  
> 1234567890abcdef  
> 0 
> some-footer: some-value  
> another-footer: another-value  
> [blank line here]  
> 
> Note the blank line after the last footer. The length of the text data is 42 bytes (1a + 10, in hex), and the data itself is abcdefghijklmnopqrstuvwxyz1234567890abcdef. The footers should be treated like headers, as if they were at the top of the response.
> 
> The chunks can contain any binary data, and may be much larger than the examples here. The size-line parameters are rarely used, but you should at least ignore them correctly. Footers are also rare, but might be appropriate for things like checksums or digital signatures.

头部\r\n\r\n结束后，就可以按chunk编码进行数据发送，每次先发送数据大小，16进制编码，使用\r\n换行,\r\n之前的非16进制编码内容会被忽略，接下来发送数据，数据可以是二进制，因为上一行已经指明的数据大小，数据发送完成使用\r\n结束当前chunk，使用0\r\n\r\n结束整个chunk编码。

chunked与multipart的区别在SO的[这个回答](http://stackoverflow.com/questions/20334859/difference-between-multipart-and-chunked-protoccol)已经讲得非常清楚

>
>Chunked is a transfer coding found in section 3.6 Transfer Codings.  
>Multipart is a media type found in section 3.7.2 Multipart Types a subsection of 3.7 Media Types.    
>...   
> This differs from the content-coding in that the transfer-coding is a property of the message, not of the entity.  
>...  
> Put more simply, chunking is how you transfer a block of data, while multipart is the shape of the data.

简单来说chunked和multipart不是一个层面的东西，transfer-encoding表明的是数据编码的方式,Content-Type则描述的是媒体的类型。

### websocket支持

Tranfer-Encoding有一种场景是实现服务器推送，服务端在一个连接中使用chunked方式进行数据编码，按一定频率发送chunk数据，但是不发送结束数据，服务器和客户端能够保持http长连接，实现服务器向客户端的数据推送。这种技术比客户端轮询进行数据更新的效率会高一些。对于服务器推送通过websocket技术，mongoose支持websocket v13，使用websocket包含必要的握手过程，流程如下：

1. 客户端和服务器建立连接
2. 客户端发送握手头
3. 服务器回应握手结果，服务端触发MG_EV_WEBSOCKET_HANDSHAKE_DONE
4. 客户端收到Sec-WebSocket-Accept:并触发MG_EV_WEBSOCKET_HANDSHAKE_DONE事件

websocket支持全双工通信，客户端和服务之间都可以调用mg_send_websocket_frame或相关函数向远端发送数据，远端在MG_EV_WEBSOCKET_FRAME事件中读取帧数据。

websocket是html5的新标准，如果在浏览器端做双向数据通信的话可以使用[socket.io](http://socket.io),socket.io保持了很好的浏览器兼容性，支持如下协议:

- WebSocket
- Adobe® Flash® Socket
- AJAX long polling
- AJAX multipart streaming
- Forever Iframe
- JSONP Polling

在支持websocket的浏览器下优先使用websocket进行通信，使用socket.io提供的nodejs包可以非常简单地构建聊天应用，socket.io的应用能力目前并没有完全被开发出来。

### 多段数据流

mongoose支持multipart streaming，编译时通过定义MG_ENABLE_HTTP_STREAMING_MULTIPART宏来开启，使用多段数据流需要在头部指定:
```
Content-Type: multipart/...; boundary=JNhbAsdujgbjhasd\r\n
```

multiparts数据使用头部指定boundary进行数据分段，没段的开始以"--${boundary}\r\n"开始,跟随一行数据处置头"Content-Disposition:...\r\n\r\n",然后是数据内容，理论上内容不能包含"--${boundary}"，以避免出现错误的数据边界识别，最后一"--${boundary}--\r\n"结束多段数据传输。

一个多段数据传输的例子：

```
Content-Type: multipart/form-data; boundary=AsdsdgsdgASAFS\r\n
\r\n
--AsdsdgsdgASAFS\r\n
Content-Disposition: name=a; filename=a.txt\r\n
\r\n
[content of a.txt]
\r\n--AsdsdgsdgASAFS\r\n
Content-Disposition: name=b; filename=b.txt\r\n
\r\n
[content of b.txt]
\r\n--AsdsdgsdgASAFS--\r\n
```

服务端识别数据为多段数据（通过头Content-Type: multipart/），首先触发服务器的MG_EV_MULTIPART_REQUEST事件，之后循环处理每个数据块，对于每个数据块，包含三个关键事件：

1. MG_EV_MULTIPART_BEGIN 段数据开始
2. MG_EV_MULTIPART_DATA 段数据内容
3. MG_EV_MULTIPART_END 段数据结束

mongoose的缺点：

1. 不够健壮，比如multipart的非法数据会导致mongoose访问越界
2. 代码质量不是很高，代码编写的不是很规范
3. 偏底层，需要熟悉http协议

mongoose有点：

1. 轻量，非常容易在宿主程序中内嵌web服务器
2. 支持websocket（支持13版本）
3. 提供大量示例，容易上手
4. 事件驱动，io复用高
5. 编程灵活性高，服务器和客户端公用一套代码库
