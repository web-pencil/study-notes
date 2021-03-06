# HTTP

## TCP连接

- HTTP协议是使用TCP作为其传输层协议的，当TCP出现瓶颈时，HTTP也会受到影响

## 协议

- http： HTTP、TCP、IP
- https: HTTP、TCP、IP、SSL(或TLS)

## TCP/IP

- TCP/IP是个协议组，可分为三个层次：网络层、传输层和应用层
  - 在网络层有IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议
  - 在传输层中有TCP协议与UDP协议
    - UDP包括DNS、TFTP等协议
  - 在应用层有:TCP包括FTP、HTTP、TELNET、SMTP等协议
- HTTP属于应用层协议，在传输层使用TCP协议，在网络层使用IP协议
  - IP协议主要解决网络路由和寻址问题，TCP协议主要解决如何在IP层之上可靠地传递数据包，使得网络上接收端收到发送端所发出的所有包，并且顺序与发送顺序一致。TCP协议是可靠的、面向连接的

## http 请求

![TCP/IP连接](/img/TCP_IP连接.png)

### 反向代理服务器

- 传统代理服务器位于浏览器一侧，代理浏览器将http请求发送到互联网上，而反向代理服务器位于网站机房一侧，代理网站web服务器接收http请求
- 当用户第一次访问静态内容的时候，静态内容就被缓存在反向代理服务器上，这样当其他用户访问该静态内容的时候，就可以直接从反向代理服务器返回，加速web请求响应速度，减轻web服务器负载压力

#### Nginx反向代理

- 当网站访问量非常大，网站越来越慢，一台服务器已经不够用了，网站可以将用户的请求先转到到Nginx反向代理服务器中
- 客户端不是直接通过HTTP协议访问某网站应用服务器，而是先请求到Nginx，Nginx再请求应用服务器，然后将结果返回给客户端
- 将同一个应用部署在多台web服务器上后，Nginx将大量用户的请求分配给多台机器处理
- 通过Nginx的反向代理，请求到达了web服务器，服务端脚本处理我们的请求，访问我们的数据库，获取需要获取的内容
- 即使其中一台服务器万一挂了，只要还有其他服务器正常运行，就不会影响用户使用
- 来自不同用户的请求 -> Nginx负载均衡系统 -> web服务器集群系统 -> 数据库集群系统

### 请求报文

- 一个HTTP请求报文由请求行（request line）、请求头（header）、空行和请求数据4个部分组成：
  - 请求方法 URL 协议版本
  - 请求头(Request Header)
  - 空行
  - 请求正文
  
- 例如：

  ```text
  GET /sample.jsp HTTP/1.1
  Accept:image/gif.image/jpeg,*/*
  Accept-Language:zh-cn
  Connection:Keep-Alive
  Host:localhost
  User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)
  Accept-Encoding:gzip,deflate

  username=jinqiao&password=1234
  ```

- 最后一个请求头之后是一个空行，发送回车符和换行符，通知服务器以下不再有请求头
- GET是请求方法，sample.jsp是URL，HTTP是协议，1.1是版本

### 请求方法

- HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法
- HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法
- 具体说明：
  - GET: 请求指定的页面信息，并返回实体主体，回送给客户端
    - GET方法要求服务器将URL定位的资源放在响应报文的数据部分
    - 利用一个问号（“?”）代表URL的结尾与请求参数的开始，各个数据之间用”&”符号隔开，会显示在URL中，所以不适合传送私密数据
    - 由于不同的浏览器对地址的字符限制也有所不同，一般最多只能识别1024个字符，所以不适合传送大量数据
  - HEAD：类似于get请求，但服务端接受到HEAD请求后只返回响应头，而不会发送响应内容
    - 当我们只需要查看某个页面的状态的时候，使用HEAD是非常高效的，因为在传输的过程中省去了页面内容
  - POST：
    - 将请求参数封装在HTTP请求数据中，以名称/值的形式出现，可以传输大量数据
    - 请求的数据保存在”请求内容”部分，具有私密性
  - PUT:
    - 从客户端向服务器传送的数据取代指定的文档的内容
  - DELETE
    - 请求服务器删除指定的页面
  - CONNECT
    - HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器
  - OPTIONS
    - 允许客户端查看服务器的性能
  - TRACE
    - 回显服务器收到的请求，主要用于测试或诊断

## http 响应

- 由三个部分组成，分别是：状态行、响应头、空行、响应正文
- 在响应中和请求的唯一区别在于状态行中的状态信息替代了请求信息
- 状态行包括：
  - 协议版本 状态码 状态的文本说明
- 例如：

  ```text
  HTTP/1.1 200 OK
  Date: Sat, 31 Dec 2005 23:59:59 GMT
  Content-type: text/html;charset=ISO-8859-1
  Content-Length: 122

  <html>
  <head>
  <title>1233</title>
  </head>
  <body>
  <!-- here -->
  </body>
  </html>
  ```

### 状态码

#### 信息性状态码 (表示服务器已接收了客户端请求，客户端可继续发送请求)

- 100  Continue
  - 继续，一般在发送post请求时，已发送了http header之后服务端将返回此信息，表示确认，之后发送具体参数信息
- 101 Switching Protocols
  - 切换协议，服务器根据客户端的请求切换协议，只能切换到更高的协议

#### 成功状态码 (表示服务器已成功接收到请求并进行处理)

- 200  OK
  - 表示客户端请求成功
- 201  Created
  - 请求成功并且服务器创建了新的资源
- 202  Accepted
  - 服务器已接受请求，但尚未处理
- 204 No Content
  - 成功，但不返回任何实体的主体部分
- 206 Partial Content
  - 成功执行了一个范围（Range）请求

#### 重定向状态码 (表示服务器要求客户端重定向)

- 301  Moved Permanently
  - 请求的网页已永久移动到新位置, 响应报文的Location首部应该有该资源的新URL
  - 搜索引擎搜索时，记录的内容和地址都是新url页面的
  - 不完整的URL都会触发重定向，包括末尾没有加/
- 302 Found
  - 临时性重定向，响应报文的Location首部给出的URL用来临时定位资源
  - 搜索引擎搜索时，记录的内容是新页面的，但是收录的地址是旧页面的，相当于新页面为旧页面的seo排名做了贡献，此处有流量劫持风险
- 303 See Other
  - 请求的资源存在着另一个URI，客户端应使用GET方法定向获取请求的资源
- 304  Not Modified
  - 服务器内容没有更新，可以直接读取浏览器缓存
- 307 Temporary Redirect
  - 临时重定向。与302 Found含义一样
  - 302禁止POST变换为GET，但实际使用时并不一定，307则更多浏览器可能会遵循这一标准，但也依赖于浏览器具体实现

#### 客户端错误状态码 (表示客户端的请求有非法内容)

- 400 Bad Request
  - 表示客户端请求有语法错误，不能被服务器所理解
- 401 Unauthorized
  - 表示请求未经授权，该状态代码必须与 WWW-Authenticate 报头域一起使用
- 403 Forbidden
  - 表示服务器收到请求，但是拒绝提供服务，通常会在响应正文中给出不提供服务的原因
- 404 Not Found
  - 找不到如何与 URI 相匹配的资源
  
#### 服务器错误状态码 (表示服务器未能正常处理客户端的请求而出现意外错误)

- 500 Internal Server Error
  - 表示服务器发生不可预期的错误，导致无法完成客户端的请求
- 503 Service Unavailable
  - 表示服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常 (可能是过载或维护)

![完整状态码](/img/http-status-code.jpg)

### 响应头

![响应头属性](/img/response-headers.jpg)

## [http1.0、http1.1、http2.0的区别](https://juejin.im/entry/6844903489596833800)

## token授权

- 发送请求时通常要在请求头中设置token
  - headers: {Authorization: `Bearer ${token}`}

