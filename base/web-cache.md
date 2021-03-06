# Web 缓存（http 缓存）

- [参考文档](https://www.oschina.net/translate/cache_docs)
- 表述：HTML 页面、图片和文件等的统称
- Web 缓存处于服务器（也称为源服务器）和客户端之间，监视请求并保存响应的表述的副本。如果之后有对同一个 URL 的新请求，它会使用自己保存的内容来响应，而不是再次请求源服务器来获取内容
- 代理和浏览器缓存默认都会被使用在某个环节中。如果你没有正确的配置站点的缓存相关配置，站点数据将会按照默认的缓存管理员的配置被缓存下来
- 缓存是请求资源的副本，要区别于cookie等前端数据存储概念
  - 广义上来讲，缓存是个概念
- 作用：
  - 缓解服务器压力
  - 减少延迟：因为响应请求的内容来自缓存（距客户端较近）而不是源服务器，它会花较少的时间来获得表述并将他们呈现出来，这使得 Web 看起来具有良好的响应速度
  - 减少网络传输：由于复用了表述，它可以减少客户端使用的带宽总量。缓存会降低对带宽的要求，也降低处理难度
  
## 缓存种类

### 浏览器缓存

- 私有缓存
- 浏览器“缓存”设置：这个选项让你配置一部分硬盘空间来保存你看过的表述
  - Chrome会根据本地内存的使用率来决定缓存存放在哪，如果内存使用率很高，放在磁盘里面，内存的使用率很低会暂时放在内存里面
  - 同一个资源有时是**from memory cache**有时是**from disk cache**
- 浏览器缓存的规则：它通常会在一次会话（即当前浏览器中第一次调用）中检查表述是否最新
- 缓存存在的意义就是当用户点击back按钮或是再次去访问某个页面的时候能够更快的响应
  - 尤其是在多页应用的网站中，如果你在多个页面使用了一张相同的图片，那么缓存这张图片就变得特别的有用
  
### 代理缓存

- 代理服务器缓存
  - 应用于同一种协议间的传输
- 规模比浏览器缓存更大，代理以同样的方式为成千上万的用户服务
- 大公司和大型的ISP提供商通常会将它们设立在防火墙上或是作为一个独立的设备（中间设备）来运营
- 代理缓存既不是客户端的一部分，也不是服务器的一部分，而是在网络之外，必须以某种方式把请求路由过去
  - 方式一：手工修改浏览器代理设备，指定要使用的代码
  - 方式二：拦截。拦截式代理会根据其自身的基础网络重定向 Web 请求，不需要在客户端配置，客户端甚至不知道它们的存在
- 代理缓存是一种共享缓存，通常不只是一个用户，而是大量用户在使用代理缓存，众人都需要的表述会被多次重复使用
  - 因此在减少相应时间和带宽使用方面很有效：因为同一个缓存可能会被重用多次
- 隧道：代理的一种延伸，对一种协议进行加密传输，比如vpn。目前协议传输较少用代理进行传输，隧道安全性高，所以较多使用
  
### 网关缓存

- 应用于不同协议间的转换传输
- 又名“反向代理缓存”
- 网关缓存服务器也是一种中介（即中间服务器），由网站管理员自己部署，使其站点更具伸缩性、可靠性以及拥有更好的性能
- 常见的方法是使用负载均衡器把请求路由到网关缓存服务器，让他们对于客户来说，看起来就跟源服务器一样
- CDNS(网络内容分发商)分布网关缓存到整个（或部分）互联网上，并出售缓存服务给需要的网站

### 数据库缓存

- 当我们的应用极其复杂，表自然也很繁杂，我们必须进行频繁的进行数据库查询，这样可能导致数据库不堪重负
  - 一个好的办法就是将查询后的数据放到内存中，下一次查询直接从内存中取就好了

## 缓存工作方式

- 所有的缓存都有一系列用来决定什么时候从缓存中提供内容的规则
- 其中的一些规则被放置在了协议中（HTTP 1.0和1.1），而另一些则由缓存的管理员（诸如浏览器缓存的用户，或者代理管理员）来设置
- 规则集
  - 如果响应的头部通知缓存不要保存当前响应内容，那么缓存就不会缓存当前响应
    - 浏览器对于缓存的处理是根据第一次请求资源时返回的响应头来确定的
  - 如果是一个授权的或者加密的请求（例如HTTPS），那么共享缓存将不会保存相关数据内容
  - 在下述场景中，我们认为被缓存的内容是最新的（意味着不需要源服务端的检查就可以被发送给客户端），故而数据内容会直接从缓存中提供且不需要源服务端的校验
    - 缓存内容由过期时间或者其他的生存期机制控制，且缓存内容仍在生存有效期内
    - 如果缓存服务近期对外提供了数据内容，且该内容在很久之前就被修改了
  - 如果内容已经过时了，源服务端会要求对其进行验证，或者通知缓存服务这份缓存的内容是否仍然有效
  - 在类似于网络中断这样的场景中，缓存可以对外提供过时的响应数据而不必和源服务器进行校验和确认
- 如果在响应中没有相应的验证器（ ETag 或者 Last-Modified 头部），且也没有明确的刷新信息，则这种数据通常被视为不可缓存的数据（但不总是）
- 刷新和验证是缓存可以正常有效的保存内容的最重要途径。新的数据内容可以快速的从缓存中得到，与此同时一个经过验证的表述则避免了在没有发生变更的情况下被再次完整的发送出去

## 各缓存参数的作用

- 当某一文件在浏览器中第一次被访问的时候，这个时候浏览器是没有缓存的，直接从服务器获取文件，返回给客户端，并且存入浏览器缓存；此时，返回状态码200，并且服务端可以设置响应头部**Expires**或者**Cache-Control**，**Last-Modified**或者**ETag**
- 如果设置了**Expires**或者**Cache-Control**，那么在指定时间内再次请求该文件时，只要不强制刷新缓存(F5等)，浏览器会直接读取缓存而不再去请求服务器
- 如果没有设置**Expires**或者**Cache-Control**或者过期了，就需要再次请求服务器了，浏览器会发起条件验证，发起请求时在请求头加上**If-Modified-Since**或者**If-None-Match**，服务器端判断最新的文件是否发生了更新，如果没有，则返回响应状态码304，并且不带任何响应实体，浏览器接受到了304响应，就知道了要读取浏览器缓存了

## 浏览器各种刷新操作的区别

- 所有操作中，报文中cache-control设置了no-cache，则每次都一定会访问服务器；设置了max-age, 则在过期之前不会重复访问
- 按回车，浏览器会判断是否有缓存，并且根据**Expires**或者**Cache-Control**判断缓存是否过期，如果没有，就不会发起请求，直接使用缓存，否则就需要向服务器发起请求再验证
- 浏览器刷新按钮和F5效果相同，不管是否有**Expires**或者**Cache-Control**，都会强制去请求服务器，进行再验证，根据**If-Modified-Since**或者**If-None-Match**判断是否要返回304，如果是，浏览器就会继续使用缓存
- 按Ctr+F5时，不管是否有**Expires**或者**Cache-Control**，都会强制去请求服务器，但是并不会进行再验证，服务器会直接把最新的内容返回给浏览器，压根就不考虑缓存的存在或者是否过期

## 缓存控制

### 报文相关字段

- 通用首部字段
  - Cache-Control
    - 控制缓存具体的行为
  - Pragma
    - 当值为"no-cache"时强制验证缓存
  - Date
    - 创建报文的日期时间(启发式缓存阶段会用到这个字段)
- 响应首部字段
  - ETag
    - 服务器生成资源的唯一标识
  - Vary
    - 代理服务器缓存的管理信息
  - Age
    - 资源在缓存代理中存贮的时长(取决于max-age和s-maxage的大小)
- 请求首部字段
  - If-Match
    - 条件请求，携带上一次请求中资源的ETag，服务器根据这个字段判断文件是否有新的修改，ETag 比较采用的是强比较算法（每个字节必须相同）
    - 在请求方法为 GET 和 HEAD 的情况下，服务器仅在请求的资源满足此首部列出的 ETag值时才会返回资源
    - 对于 PUT 或其他非安全方法来说，只有在满足条件的情况下才可以将资源上传
  - If-None-Match
    - 和If-Match作用相反，服务器根据这个字段判断文件是否有新的修改，ETag 比较采用的是弱比较算法
    - 只有当没有资源的 **ETag** 相匹配的情况下返回200或对请求进行相应的处理
      - 对于 GET 和 HEAD 方法来说，当验证失败的时候，服务器端必须返回响应码 304
      - 对于能够引发服务器状态改变的方法，当验证失败的时候则返回 412 （Precondition Failed，前置条件失败）
    - 优先级比 **If-Modified-Since** 更高
  - If-Modified-Since
    - 比较资源前后两次访问最后的修改时间是否一致
    - 如果有修改，正常返回资源，状态码200
    - 如果没有修改，只返回响应头，状态码304，告知浏览器资源的本地缓存还可用
  - If-Unmodified-Since
    - 比较资源前后两次访问最后的修改时间是否一致
    - 如果文件修改了则返回状态码412(预处理错误)
    - 如果没有被修改则返回200和资源
- 实体首部字段
  - Expires
    - 告知客户端资源缓存失效的绝对时间
  - Last-Modified
    - 资源最后一次修改的时间

### HTML Meta 标签

- meta 标签常用于确保文档不被缓存，或者在一定时间后过期。但效果不怎么样

```html
<meta http-equiv="expires" content="Thu, 30 Nov 2017 11:17:26 GMT">


<meta http-equiv="Pragma" content="no-cache">
<!-- 会在请求头上设置 Pragma: no-cache -->
<!-- 仅有IE才能识别这段meta标签含义，其它主流浏览器仅能识别Cache-Control: no-store的meta标签
在IE中识别到该meta标签含义，并不一定会在请求字段加上Pragma，但的确会让当前页面每次都发新请求(仅限页面，页面上的资源则不受影响) -->
```

- 只有部分浏览器缓存会遵从约定，代理缓存却不会（代理基本上不会去分析文档中的 HTML）

#### Pragma HTTP 头

- HTTP/1.0中的定义，是通用首部字段，优先级很高
  - Chrome 和 Firefox 中 Pragma 的优先级高于Cache-Control和Expires
- 当值为"no-cache"时强制验证缓存
  - 指定 Pragma: no-cache HTTP 头后，表述依旧可能被缓存
  - 不是指的Cache-Control HTTP 头的 no-cache
- 服务端响应添加'Pragma': 'no-cache'，浏览器表现行为和强制刷新类似

#### Expires HTTP 头控制新近程度

- HTTP/1.0中的定义缓存的字段，存在响应头中
- Expires HTTP 头是控制缓存的基础方法，它告诉所有缓存与之相关的表述存在多久的保鲜期
- 保鲜期之后，缓存应该检查源服务器，看文档是否被改变。几乎各种缓存都支持 Expires 头
- 设置方法：
  - 设置绝对的过期时间，根据上次客户端取回表述时（最近访问时间）计算的时间
  - 根据上次服务器文档修改时间计算的时间（最近修改时间）
- Expires 头特别适合缓存静态图像（比如导航栏和按钮），因为他们不会经常变化，你可以为他们设置一个非常长的过期时间，使你的站点具有更优势的响应性能
- 根据页面信息更新规律设置缓存更新时长和时间点
- Expires 头只支持 HTTP 日期值，任何其它值都会被认为“过去时”，结果表述不会被缓存
- HTTP 日期是格林威治（GMT）时间，而不是本地时间。如 Expires: Thu, 15 Apr 2010 20:00:00 GMT
- Expires有一个很大的弊端，就是它返回的是服务器的时间，但判断的时候用的却是客户端的时间，这就导致Expires很被动，因为用户有可能改变客户端的时间，导致缓存时间判断出错，这也是引入Cache-Control:max-age指令的原因之一
- 可以使用网络时间协议（NTP）
- 局限：
  - Web 服务器和缓存上的时钟就必须同步
  - 必须保证服务器时钟的准确性
  - 容易忘记为某些内容设置特定的过期时间
  
#### Cache-Control HTTP 头

- 当 Cache-Control 和 Expires 的优先级（Cache-Control都是 HTTP/1.1定义的关于缓存的字段）
  - 当 Expires 和 Cache-Control:max-age=xxx 同时存在的时候取决于缓存服务器应用的HTTP版本
    - 应用HTTP/1.0版本的缓存服务器则会优先处理Expires而忽略max-age
    - 应用HTTP/1.1版本的服务器会优先处理max-age，忽略Expires
- Cache-Control 响应头，控制缓存具体的行为
  - 让 Web 可以更方便地控制内容，避免 Expires 所具有的限制
- 请求指令
  - no-cache
    - 强制源服务器再次验证（注意no-store才是不让缓存）
    - 设置 max-age=0 和 no-cache 的结果是一样的，但是语义是不一样的
  - no-store
    - 不缓存请求或是响应的任何内容
  - max-age=[秒]
    - 缓存的时长，也是响应的最大的Age值
  - min-fresh=[秒]
    - 期望在指定时间内响应仍然有效
  - no-transform
    - 代理不可更改媒体类型
  - only-if-cached
    - 从缓存获取
  - cache-extension
    - 新的指令标记(token)
- 响应指令
  - public
    - 响应可以被任何对象（包括：发送请求的客户端，代理服务器，等等）缓存，即使是通常不可缓存的内容（例如，该响应没有max-age指令或Expires消息头）
  - private
    - 可省略
    - 响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）。私有缓存可以缓存响应内容
  - no-cache
    - 可省略
    - 缓存前必须先确认其有效性（注意no-store才是不让缓存）
  - no-store
    - 不缓存请求或响应的任何内容
  - no-transform
    - 代理不可更改媒体类型
  - must-revalidate
    - 可缓存但必须再向源服务器进确认
    - HTTP 允许缓存服务在一些特殊情况下认为表述过期，你可以通过指定这个头参数告诉缓存你希望它严格遵守你的规则
  - proxy-revalidate
    - 要求中间缓存服务器对缓存的响应有效性再进行确认
    - 与 must-revalidate 相似，但它只对代理缓存有效
  - max-age=[秒]
    - 缓存的时长，也是响应的最大的Age值
    - 是从请求开始你期望表述过期前保持有效的总秒数
  - s-maxage=[秒]
    - 共享缓存服务器响应的最大Age值
    - 和 max-age 相似，但它只对共享（例如代理）缓存有效
  - cache-extension
    - 新的指令标记(token)

### Last-Modified

- 没法准确的判断资源是否真的修改了
  - 比如某个文件在1秒内频繁更改了多次，根据Last-Modified的时间(单位是秒)是判断不出来的
  - 某个资源只是修改了，但实际内容并没有发生变化，Last-Modified也无法判断出来
- 由于其局限性，HTTP/1.1中还推出了 **ETag** 这个字段

### ETag

- 服务器可以通过某种自定的算法对资源生成一个唯一的标识(比如md5标识)，然后在浏览器第一次请求某一个URL时把这个标识放到响应头传到客户端。服务器端的返回状态会是200
- ETag的值有可能包含一个 W/ 前缀，来提示应该采用弱比较算法

### If-Match

- 对于 GET 和 HEAD 方法，搭配 Range首部使用，可以用来保证新请求的范围与之前请求的范围是对同一份资源的请求，如果 ETag 无法匹配，那么需要返回 416 (Range Not Satisfiable，范围请求无法满足) 响应
- 对于其他方法来说，尤其是 PUT, If-Match 首部可以用来检测用户想要上传的不会覆盖获取原始资源之后做出的更新（更新丢失问题）
  - 如果请求的条件不满足，那么需要返回 412 (Precondition Failed，先决条件失败) 响应

### If-None-Match

- 采用 GET 或 HEAD 方法，来更新拥有特定的 ETag 属性值的缓存
- 采用其他方法，尤其是 PUT，将 **If-None-Match** used 的值设置为 * ，用来生成事先并不知道是否存在的文件，可以确保先前并没有进行过类似的上传操作，防止之前操作数据的丢失（更新丢失问题）

### If-Unmodified-Since

- 与含有 **If-Range** 消息头的范围请求搭配使用，实现断点续传的功能，即如果资源没修改继续下载，如果资源修改了，续传的意义就没有了
- POST、PUT请求中，优化并发控制，即当多用户编辑同一份文档的时候，如果服务器的资源已经被修改，那么在对其作出编辑会被拒绝提交

### Range

- `Range: <unit>=<range-start>-<range-end>, <range-start>-<range-end>, <range-start>-<range-end>`
  - `<unit>`
    - 范围所采用的单位，通常是字节（bytes）
  - `<range-start>`
    - 一个整数，表示在特定单位下，范围的起始值
  - `<range-end>`
    - 一个整数，表示在特定单位下，范围的结束值
    - 可选，如果不存在，表示此范围一直延伸到文档结束
  - Range: bytes=200-1000, 2000-6576, 19000-
- 是一个请求首部，告知服务器返回文件的哪一部分
- 在一个 Range 首部中，可以一次性请求多个部分，服务器会以 multipart 文件的形式将其返回
- 如果所请求的范围成功，服务器返回的是范围响应，需要使用 206 状态码
- 假如所请求的范围不合法，那么服务器会返回  416 状态码，表示客户端错误
- 如果所请求的范围不成功，服务器允许忽略 Range 首部，从而返回整个文件，状态码用 200

### If-Range

- HTTP 请求头字段用来使得 Range 头字段在一定条件下起作用
- 当字段值中的条件得到满足时，Range 头字段才会起作用，同时服务器回复 206(部分内容状态码)，以及 Range 头字段请求的相应部分
- 如果字段值中的条件没有得到满足，服务器将会返回 200 OK 状态码，并返回完整的请求资源
- 字段值中既可以用 Last-Modified 时间值用作验证，也可以用ETag标记作为验证，但不能将两者同时使用
- If-Range 头字段通常用于断点续传的下载过程中，用来自从上次中断后，确保下载的资源没有发生改变
- If-Range: Wed, 21 Oct 2015 07:28:00 GMT
- If-Range: 675af34563dc-tr34

### 三种缓存阶段

- 强缓存阶段
  - 使用了 **Expires** 字段，当浏览器再次试图访问这个文件，发现有这个文件的缓存，那么就判断根据上一次的响应判断是否过期，如果没过期，使用缓存，加载文件
- 协商（弱）缓存阶段
  - 当浏览器再次试图访问这个文件，发现缓存（Expires）过期，于是会在本次请求的请求头里携带**If-Modified-Since**和**If-None-Match**（可能有）这两个字段，服务器通过这两个字段来判断资源是否有修改
    - **If-Modified-Since**的值用的是 **Last-Modified** 的值
    - **If-None-Match** 的值用的是 **ETag**
    - 如果有修改则返回状态码200和新的内容
      - 浏览器收到200状态码，按首次访问此文件的情况进行处理
    - 如果没有修改返回状态码304
      - 浏览器收到304，于是知道了本地缓存虽然过期但仍然可以用，于是加载本地缓存，然后根据新的返回的响应头来设置缓存
      - 这一步有所差异，发现不同浏览器的处理是不同的，chrome会为304设置缓存，firefox则不会
- 启发式缓存阶段
  - 当响应头没有明确的缓存过期时间（Expires）字段，浏览器的下次请求不会直接进入协商缓存阶段，而是根据响应头中2个时间字段 **Date** 和 **Last-Modified** 之间的时间差值，取其值的10%作为缓存时间周期

### 校验器和校验

- 存在响应头中
- 在表示层发生变化时服务器和缓存使用校验进行通信
- 通过使用校验，缓存可以避免在本地已有副本时下载整个表示层。当他们不确定它是否仍然是最新的，所以需要校验器去校验
- 如果校验器不存在，并且没有任何新的信息（ Expires 或 Cache-Control ）可用，则缓存将根本不存储表示层
- 常用校验器
  - Last-Modified：最通用的校验器是头部的 Last-Modified，用来标识文档最近一次修改的时间
    - 如果缓存存储了一个带有Last-Modified 头部的表示层，缓存可以借助一个 If-Modified-Since 请求向服务端确认当前缓存的表示层在最近一次修改后是否发生了变更
  - ETag：ETags是一个由服务端生成的唯一标识符，并且每当表示层发生变更时ETags的值都会发生变化
    - 由于ETag是由服务端生成的，所以当缓存通过 If-None-Match 请求得知ETag在服务端匹配成功时，便可以确认缓存存储的表示层和服务端的内容是一致的，没有发生任何变化
- 几乎所有的缓存都使用了最近一次修改时间来作为校验器，同时ETag校验器的使用也在逐步增长
  - 大多数的现代网站服务器会自动地为静态内容（例如文件）同时生成 ETag 和 Last-Modified 这两个校验器，这个过程不需要任何人为参与。然而，服务器在为诸如CGI、ASP或者数据库站点这样的动态内容生成 ETag 和 Last-Modified 校验器时就显得力不从心了
- 同时使用**Last-Modified**和**ETag**实体标签进行验证的原因：
  - 有些文档会被周期性的重写，但实际包含的数据是一样的（尽管内容没有变化，最后修改日期却会发生变化）
  - 有些文档可能被修改了，但是修改并不重要，没必要更新缓存
  - 有些服务器无法准确判定页面的最后修改日期
  - 文档在毫秒级间隙发生变化（如实时监控），以秒为颗粒度的**Last-Modified**就不够用了

### 强弱校验

| ETag1 | ETag2 | Strong Comparison | Weak Comparison |
| :---: | :---: | :---------------: | :-------------: |
| W/"1" | W/"1" | no match          | match           |
| W/"1" | W/"2" | no match          | no match        |
| W/"1" | "1"   | no match          | match           |
| "1"   | "1"   | match             | match           |

### 缓存服务器相关的两个字段

- Vary
  - 可以使用 Vary: User-Agent 来区分不同的客户端
  - 源服务器启用了gzip压缩，但用户使用了比较旧的浏览器，不支持压缩，则可以设置 Vary: Accept-Encoding
  - 也可以同时使用多个设置 Vary: User-Agent, Accept-Encoding
- Age
  - 指资源在缓存服务器存在的时长，用来区分请求的资源来自源服务器还是缓存服务器的缓存的
    - Cache-Control: max-age=[秒] 就是Age的最大值
  - 需要结合 Date 字段进行判断

  ```text
  Accept-Ranges: bytes
  Age: 1016859
  Cache-Control: max-age=2592000
  Content-Length: 14119
  Content-Type: image/png
  Date: Fri, 01 Dec 2017 12:27:25 GMT
  ETag: "5912bfd0-3727"
  Expires: Tue, 19 Dec 2017 17:59:46 GMT
  Last-Modified: Wed, 10 May 2017 07:22:56 GMT
  Ohc-Response-Time: 1 0 0 0 0 0
  Server: bfe/1.0.8.13-sslpool-patch

  例如有这样一个响应头，按F5频繁刷新发现响应里的Date没有改变，就说明命中了缓存服务器的缓存
  这个资源已经在缓存服务器存在了1016859秒。如果文件被修改或替换，Age会重新由0开始累计
  Age消息头的值通常接近于0。表示此消息对象刚刚从原始服务器获取不久；其他的值则是表示代理服务器当前的系统时间与此应答消息中的通用消息头 Date 的值之差
  静态资源Age + 静态资源Date = 原服务端Date
  ```

### 构建缓存感知站点的技巧

- 除了以上使用新鲜度信息和校验，以下方法值得参考
  - 始终使用同一url：如果向不同的页面、不同的用户提供同样的数据内容，或者同样的内容来自于不同的站点，这时应该使用一致的URL
    - 缓存的黄金原则
    - 最简单、最有效的让站点更利于缓存的方法
  - 使用一个包含图片和其他元素的公共库，并在不同的地方引用他们
  - 使用缓存来存储图片和很少变更的页面
    - 这个的实现可以借助一个设置了很大的值的 Cache-Control: max-age 头部信息
  - 通过一个精确的最大存活时间或者过期时间让缓存识别出更新频繁的页面
  - 如果资源（特别是可下载的文件）发生了变更，改变其名字
    - 通过这种方式，可以让资源在未来的某个时间过期，同时也能保证当前版本仍然是有效的。唯一需要设置一个短的过期时间的部分就是链接到这些资源的页面
  - 在必要的情况下再修改文件
    - 如果非必要的情况下也选择更新文件，那么每个文件都会有一个不真实的距离当前时间更近的 Last-Modified 值。
    - 例如当准备更新站点时，不要复制整个站点文件进行更新，仅选择那些确实修改了的文件去执行更新操作
  - 只在有需要的情况下使用cookie
    - cookies很难被缓存存储起来，并且在大多数场景都是没有必要的
    - 如果必须用到cookie的话，那么也只在动态页面中使用cookie

## 编写缓存可感知的脚本

- 如果脚本生成的输出在以后的某个时间（无论是几分钟还是几天后）都可以使用相同的请求重现，那么它应该是可缓存的
- 如果脚本的内容仅根据 URL 中的内容而变动，则它是可缓存的
- 如果其输出取决于cookie、身份认证信息或其他外部标准，则它可能不是可缓存的
- 技巧：
  - 让脚本对缓存友好（同时有更好的表现）的最佳方式是只要脚本发生变化，就将其内容转储到一个普通文件中
    - Web 服务器会把这个文件跟其它 Web 页面同等对待，为其生成验证器，让一切变得简单
    - 只写内容变动过的文件，避免刷新新没有内容变动文件的 Last-Modified 时间
  - 设置一个跟寿命相关的头，可以让脚本在一定的限制条件下被缓存。
    - 虽然用 Expires 可以做到，但用 Cache-Control: max-age 可能更简单，它会按一定时间间隔刷新请求
  - 用脚本生成一个验证器，然后响应 If-Modified-Since 或 If-None-Match 请求
    - 这个操作可以通过解析 HTTP 头之后，适当的响应 304 Not Modified 来实现
  - 不使用 POST，除非确有必要
    - 多数缓存不会保存 POST 响应。如果你通过路径或查询（通过 GET）发送信息，缓存会保存这些信息以备将来使用
  - 不要在 URL 中嵌入用户特定的信息，除非生成的内容对每个用户都不同
  - 不要以为所有用户请求都来自同一台主机，因为也有可能来自缓存
  - 生成 Content-Length 响应头，它会允许通过长连接来响应脚本
    - 客户端可以在一个 TCP/IP 连接上请求多个表述，而不是为每个请求建立连接。这样网站看起来会更快
  
## 例子

- post请求拉取大量数据的缓存策略

![post请求拉取大量数据的缓存策略](/img/post-cache.png)
