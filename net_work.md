# TCP/IP
- TCP 协议的全称是 Transmission Control Protocol 的缩写，意思是传输控制协议
- IP 协议的全称是 Internet Protocol 的缩写
DNS 的全称是域名系统（Domain Name System，缩写：DNS）

## 三次握手四次挥手
### 为什么需要三次握手，两次不行吗？
试想如果是用两次握手，则会出现下面这种情况：

如客户端发出连接请求，但因连接请求报文丢失而未收到确认，于是客户端再重传一次连接请求。后来收到了确认，建立了连接。数据传输完毕后，就释放了连接，客户端共发出了两个连接请求报文段，其中第一个丢失，第二个到达了服务端，但是第一个丢失的报文段只是在某些网络结点长时间滞留了，延误到连接释放以后的某个时间才到达服务端，此时服务端误认为客户端又发出一次新的连接请求，于是就向客户端发出确认报文段，同意建立连接，不采用三次握手，只要服务端发出确认，就建立新的连接了，此时客户端忽略服务端发来的确认，也不发送数据，则服务端一致等待客户端发送数据，浪费资源。
### 什么是半连接队列？
服务器第一次收到客户端的 SYN 之后，就会处于 SYN_RCVD 状态，此时双方还没有完全建立其连接，服务器会把此种状态下请求连接放在一个队列里，我们把这种队列称之为半连接队列。

### 什么是SYN攻击
SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，Server不断重发直至超时，这些伪造的SYN包将长时间占用半连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。SYN 攻击是一种典型的 DoS/DDoS 攻击
使用netstat可以检测半连接状态
```netstat -n -p TCP | grep SYN_RECV```

### 挥手为什么需要四次
也可以三次，但是关闭连接时，当服务端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉客户端，"你发的FIN报文我收到了"。只有等到我服务端所有的报文都发送完了，我才能发送FIN报文。
先回复ACK，避免客户端重复发送FIN

### TIMEWAIT状态 等待2MSL的意义
TIME_WAIT状态就是用来重发可能丢失的ACK报文
1. 为了保证客户端发送的最后一个ACK报文段能够到达服务器。因为这个ACK有可能丢失，从而导致处在LAST-ACK状态的服务器收不到对FIN-ACK的确认报文。服务器会超时重传这个FIN-ACK，接着客户端再重传一次确认，重新启动时间等待计时器。最后客户端和服务器都能正常的关闭。假设客户端不等待2MSL，而是在发送完ACK之后直接释放关闭，一但这个ACK丢失的话，服务器就无法正常的进入关闭连接状态。
2. 客户端在发送完最后一个ACK报文段后，再经过2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失，使下一个新的连接中不会出现这种旧的连接请求报文段。

### close wait 
close wait 极有可能是服务器本身的代码bug，因为服务端处理完自身逻辑后要发送syn，才会从close wait转变到last ack

## TCP/UDP区别
1. TCP是有链接的，UDP是无连接的
2. 由于有链接，TCP维护了连接状态等，因此要求系统资源比较多
3. TCP面向字节流，UDP面向报文
4. TCP可靠传输，有流量控制和拥塞控制


# HTTP
## HTTP 请求相应过程（浏览器输入网址后发生的事情）
假设访问http://wwww.example.com/index.html，输入网址回车后
1. DNS服务器首先进行域名的映射，找到www.example.com所在的地址
2. 浏览器在80客户端发起到example的TCP连接，3次握手，客户端和服务端进程中都会有一个socket
3. 客户端通过套接字发送http请求报文
4. 服务器接受报文，解析并进行处理，组装成响应报文返回给客户端
5. 服务器通知断开TCP连接，四次挥手
6. 客户端解析报文，渲染html

## GET POST区别
粗略的讲：
- GET 用于获取信息，是无副作用的，是幂等的，且可缓存
- POST 用于修改服务器上的数据，有副作用，非幂等，不可缓存

实际上：
- 在约定中，GET 方法的参数应该放在 url 中，POST 方法参数应该放在 body 中，但两种方法本质上是 TCP 连接，没有差别，也就是说，如果我不按规范来也是可以的。我们可以在 URL 上写参数，然后方法使用 POST；也可以在 Body 写参数，然后方法使用 GET。当然，这需要服务端支持。
- 我们可以自己约定参数的写法，只要服务端能够解释出来就行
- POST 比 GET 安全，因为数据在地址栏上不可见？
然而，从传输的角度来说，他们都是不安全的，因为 HTTP 在网络上是明文传输的，只要在网络节点上捉包，就能完整地获取数据报文。
要想安全传输，就只有加密，也就是 HTTPS
- HTTP 协议没有 Body 和 URL 的长度限制，对 GET请求2048的URL长度 限制的大多是浏览器和服务器的原因。
- POST 方法会产生两个 TCP 数据包？是部分浏览器或框架的请求方法，不属于 post 必然行为

## CORS 跨域
cross-origin resource sharing is an HTTP-header based mechanism that allows a server to indicate any other origins than its own from which a browser should permit loading of resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.

An example of a cross-origin request: the front-end JavaScript code served from https://domain-a.com uses XMLHttpRequest to make a request for https://domain-b.com/data.json.
### Origin
Web 概念中域(Origin) 的内容由scheme(protocol) - 协议，host(domain) - 主机和用于访问它的 URL port - 端口定义。仅仅当 scheme 、host、port 都匹配时，两个对象才有相同的来源。

> (1) http://example.com/app1/index.html
> (2) http://example.com/app2/index.html

这两个url就是同域的

>http://example.com/app1
>https://example.com/app2

这两个URL就是跨域的，协议不同

>http://example.com
>http://www.example.com
>http://myapp.example.com

这三个URL是跨域的，主机不同

>http://example.com
>http://example.com:8080

这两个url是跨域的，端口不同

### 如何控制跨域
服务端在response头中添加`Access-Control-Allow-Origin: *`， * 代表该资源可以被任何域访问
`Access-Control-Allow-Origin: https://foo.example` 则只允许 https://foo.example 访问

# cookie session jwt
## JWT
Json Web Token,JWT 是能够安全传输信息的一种方式。通过使用公钥/私钥对 JWT 进行签名认证
## jwt 和session的不同
- jwt具有加密签名
- session cookie可能存在禁止跨域问题，jwt可跨域


# socket编程
## IO 模式
对于一次IO访问，数据会先被拷贝到操作系统内核的缓冲区，然后从缓冲区拷贝到应用程序的地址空间。
因此linux产生了五种网络模式
- 阻塞I/O blocking IO
- 非阻塞I/O nonblocking IO
- I/O多路复用 IO multiplexing
- 异步I/O asynchronous IO

## accept / recv 
这两个函数不是可互换的
accept用于监听socket的第一次连接，然后返回一个新的socket，使用这个新的socket来进行首发数据


## select
select 本质上是通过
- 每次调用select，都需要把fd_set 从用户态拷贝到内核态，描述符很多时开销大
- 可监控的文件描述符个数取决与sizeof(fd_set)的值，一般是fd_set 是int32[32]的数组。
- select返回的是有事件的描述符数量，要遍历fd_set才知道那些描述符发生了事件
- select的触发是水平触发，没有事件的描述符清0，如果应用没有对有时间的描述符进行IO操作，那么之后每次select调用这些事件描述符不会清0