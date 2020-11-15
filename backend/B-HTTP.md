前端 ： 

缓存

\- Cache-Controll maxAge=xxx

\- 客户端缓存

\- 代理服务器缓存

\- 如何验证缓存的可用性





最简单的例子 ： 

```
- 输入 URL 打开网页 - AJAX 获取数据 - img 标签加载图片
```


Cache-Control

\- max-age=100 :  s

\- public、private ： 控制 ： 只能在客户端缓存，是否可以在代理服务器缓存

\- must-revalidate ： 缓存过期策略 ： 过期后必须到服务器端进行验证，才能继续使用缓存

\- no-cache、no-store ： 控制是否使用缓存





缓存验证 ： 保存在客户端

\- last-modified + if-modified-since

\- etag + if-none-match ： 





缓存是 web 服务中性能提升最大的一块





其他头信息 ： 

Content-Type、Content-Encoding ： 约束传输类型

Cookie ： 会话信息

CORS ： 跨域访问，并保证安全性限制







深入 Http ： 

```
什么是三次握手 HTTPS 链接的创建过程，以及为什么 HTTPS 就是安全的 什么是长链接，为什么需要长链接 HTTP2 的信道复用又为什么能提高性能
```



深入 TCP ： 





带来什么 ？

```
对于后端开发同学，你能够打造性能更好的 HTTP 服务 对于前端开发同学，你能够好得使用 HTTP 的特性帮助你进行开发 能够帮助前后端更好得协作
```

# 基本

组成方式：

请求报文包含三部分：
a、请求行：包含请求方法、URI、HTTP版本信息
b、请求首部字段
c、请求内容实体
响应报文包含三部分：
a、状态行：包含HTTP版本、状态码、状态码的原因短语
b、响应首部字段
c、响应内容实体、


**URI**

**（1） URI** 

统一资源标志符，包含 URL、 URN

**（2） URL** 

Uniform Rsource Locator 统一资源定位器

path : 路由定位，用于辨别当前 URL 所要请求的数据，在程序中判断而非对应目录映射

hash : 文章中的某个片段， FE 中作为页面 【锚点】定位工具

```
http://user:pass@host.com:80/path?query=string#hash
```

**（3） URN** 

永久统一资源定位符

\- 解决同一个 URN 访问资源，即使资源搬离原来的位置(URL?)，避免 404 状态码

> 在资源移动之后还能被找到 目前还没有非常成熟的使用方案




## Method

HTTP 方法 ： 用于实现 RESTful 风格的 API 访问

用来定义对资源的操作

\- PUT、DELETE、 GET、POST : 定义的语义

\- PATCH、HEAD



**GET**

获取资源



**POST**

传输实体数据



**PUT**

传输文件，不会进行认证；



**DELETE**



**HEAD**

获取报文首部，用于验证 URI 是否有效，安全、查询方法；



***OPTIONS**

查询 URI 支持的 HTTP 方法



## Statue Code

<p align="center">状态码类别</p>

|      | 类别                            | 原因短语                   |
| ---- | ------------------------------- | -------------------------- |
| 1XX  | Informational (信息性状态码)    | 接收的请求正在处理         |
| 2XX  | Success (成功状态码)            | 请求正常处理完毕           |
| 3XX  | Redirection (重定向状态码)      | 需要进行附加操作以完成请求 |
| 4XX  | Client Error (客户端错误状态码) | 服务器无法处理请求         |
| 5XX  | Server Error (服务器错误状态码) | 服务器处理请求出错         |

<p align="center">常见状态码</p>

|      | 说明                                                      | 备注                                                         |
| ---- | --------------------------------------------------------- | ------------------------------------------------------------ |
| 100  | HTTP1.1 中控制只发送 Header，在之后才发送 Body 中的内容； | 接收的请求正在处理                                           |
| 302  | 临时性重定向                                              | 服务器返回的头部信息中会包含一个 Location 字段，内容是重定向到的url。 |
| 304  |                                                           |                                                              |
| 400  |                                                           |                                                              |
| 401  |                                                           |                                                              |
| 403  |                                                           |                                                              |
| 404  |                                                           |                                                              |
| 500  |                                                           |                                                              |
| 501  |                                                           |                                                              |







## 机制

HTTP 整个流程步骤

1. 读取缓存，若 cache 没过期则直接显示，否则重新请求；
2. 解析 DNS ，域名 ==>> ip                  // 网络层
3. 建立 TCP 连接，三次握手；               // 传输层
4. 发送 Http 请求；                                                    // 应用层
5. Web 服务器处理请求并返回；
6. 加载页面；
7. 关闭 TCP 连接





**域名解析过程**


# Base And Develop

底三层 ： 

> 物理层主要作用是定义物理设备如何传输数据 数据链路层在通信的实体间建立数据链路连接 网络层为数据在结点之间传输创建逻辑链路





传输层 ： 

>  向用户提供可靠的【端到端】（End一to一End）服务 分层架构 ： 传输层向高层【屏蔽】了下层数据通信的细节

分片、可靠





应用层 ： 

> 为应用软件提供了很多服务 构建于 TCP 协议之上 分层架构 ： 屏蔽网络传输相关细节





**HtTTP/0.9~2.0**

Http/0.9

- 只有一个命令GET
- 没有 HEADER 等描述数据的信息 
- 服务器发送完毕，就关闭 【TCP连接】





4 大 ： Method、Status Code、Header、Cache



Http/1.0``增加了很多命令, POST、PUT、HEAD 增加 status code和 header 多字符集支持、多部分发送、权限、缓存等



**HTTP 协议特征**

支持 C/S 模式；

简单快速、灵活；

无连接；

无状态；



## 连接

**（1） 连接的分类**

（1） 短连接

一个 TCP 连接一次 HTTP 请求

（2） 长连接

（3） 管道连接





**长连接**

只需要建立一次 TCP 连接就能进行多次 HTTP 通信；

HTTP1.1 默认长连接，HTTP1.0 需要显视指定字段实现；

```
# 1.0
Connection: close
# 1.1
Connection: Keep-Alive
```



**流水线**

（1） 默认

HTTP 请求按序发送，下一个请求只有在当前请求收到响应之后才会发出。

可能因网络延迟和带宽问题，在下一个请求被发送到 server 之前等待很长时间。



（2） pipeline 的实现

在同一条长连接上发送连续的请求，不用等待响应返回，避免连接延迟。



并发连接数 VS 开销

复用 TCP/IP 连接



长连接的 timeout： 自动关闭

一般保存长连接



Connection: keep-alive



Chrome 并发为 6 的连接







**信道复用**

Http/2

信道复用： 在 TCP 连接上，可以并发的去发送 http 请求，通过一个 TCP 连接访问网站

​    |-- 限定，『同域』时才为同一个 TCP 连接





Google 实现 http/2





## HTTP1.1

**Http1.0 VS HTTP1.1**

① HTTP1.1 默认使用长连接，支持流水线，而 HTTP1.0 需要显示的通过在田间 `Connection: keep-alive` 实现；

② 支持只发送 Header，通过返回状态码 100 来允许发送 Body 信息；

③ 支持虚拟域名，能够使用一台物理机对应多个子域名；







**长连接**

默认情况下使用长连接，

在 HTTP1.0 时需要显示的进行使用长连接，在 HTTP1.1 需要可显示的不使用长连接



**支持流水线**



**虚拟域名**

支持通过一台物理机对应多个子域名；







**状态码与 header**

支持只发送 header信息，通过 状态码 100 来允许发送 body 信息；



**新增缓存处理指令 max-age**





HTTP/1.1 ： 目前广泛，通过字符串传输

持久连接 pipeline ： 串行 和 并行， 一个耗时长的，后面跟了一个耗时短的 增加 host 和其他一些命令 ： 通过 host 实现在一台物理机器上跑多个不同的 web 服务， 通过 host 字段表示请求道该台物理服务器上``    nodejs | java， 提高物理服务的效率.





## HTTP2.0

**HTTP1.x 缺陷**

客户端需要多个连接才能实现并发和缩短延迟；

不会压缩请求和响应首部，从而导致不必要的网络流量；

不支持有效的资源优先级，致使底层 TCP 连接的利用率低下。





**二进制分帧层**

分为 HEADERS 帧和 DATA 帧，都是二进制格式的；

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

- 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
- 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。
- 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。



**header 压缩**

客户端和服务端同时维护和更新一个包含之前见过的首部字段，避免重复传输；

且通过 Huffman 编码对首部字段进行压缩；



**服务器端推送**

客户端请求一个资源时，会把相关资源一起发送给客户端，如请求一个 html，将对应的 js, css 相关资源一起发送给客户端。





**HTTP/2.0  VS HTTP1.0**

① 二进制分帧层： 所有数据以**二进制传输**,  同一个连接里面发送多个请求不再需要按照顺序来;

③ header 压缩： 利用 HPACK 对消息头进行压缩传输，客服端和服务器维护一个动态链表（当一个头部没有出现的时候，就插入，已经出现了就用表中的索引值进行替代），将既避免了重复 header  的传输，又减小了需要传输的大小

④ 服务器端推送：  客户端请求html 的时候，服务器顺带把此html 需要的css,js
也一起发送给客服端。

② 多路复用： 即连接共享，建立起一个连接请求后，可以在这个链接上一直发送，不
要等待上一次发送完并且受到回复后才能发送下一个（http1.0 是这样），是可以同时发送多个请求，互相并不干扰。



## HTTPS

**（1） HTTP 的缺点**

- 明文传输，可能被窃取窃听(抓包分析)，==>MD5;
- 不会验证通信方的身份，可能遭遇伪装， ==> 数字签名；
- 无法证明报文的完整性，可能遭到篡改，==>  hash 



（2） 原理

在 Client -- Server 加一条 SSL 安全通道，进行通信加密 +安全证书进行安全性完整性保护。





**加密**

（1） 对称和非对称加密

非对称加密速度慢，但可以更安全地将公开密钥传输给通信发送方；

对称加密运算速度快，无法安全地将密钥传输给通信方；



（2） HTTPS 中的加密

采用混合的加密方式，使用非对称秘钥加密用于传输对称秘钥保证传输过程安全，

之后通过对称秘钥加密进行通信保证通信过程的效率



**认证**

通过 证书 来对通信方进行认证；

（1） 数字证书认证机构(CA)

为客户端和服务器端双方都可信赖的第三方机构



作用： 

服务端将公开证书发给客户端；

客户端使用数字签名进行验证；



（2） 使用流程

① 运营人员向 CA 提出公开密钥申请，

② CA 判明申请者身份后，对已申请的公开密钥做数字签名，分配这个已签名的公开密钥，将该密钥放入公开密钥证书后绑定在一起。

③ 进行 HTTPS 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先 <u>使用数字签名进行验证</u> ，如果验证通过，就可以开始通信了。



**完整性保护**

SSL 提供 报文摘要 功能来进行完整性保护。

（1） 安全原理

结合了加密和认证两个操作，基于加密后的报文遭到篡改后很难重新计算报文摘要，无法轻易获取明文。





**缺点**

① 速度： 加密和解密流程造成速度慢；

② 成本： 支付给 CA 的高额费用；





**HTTP 与  HTTPS 的区别**

HTTPS 需要 CA 证书， HTTP 不需要；

HTTPS 密文传输，HTTP 明文传输；

连接方式不同， HTTPS 使用 443 端口， HTTP 使用 80 端口；

HTTP = HTTP + 加密 + 认证 + 完整性保护，叫 HTTP 安全；





## 转发 | 重定向

转发和重定向的区别？

转发为服务器内部的行为，路径地址不会改变，中间将自己的 requst 

重定向为客户端的行为





资源搬离问题： 

302 + Location







## 三次握手

Http :  只有请求 、响应包

TCP ： 提供传输的通道

http/1.0 通过声明方式保存连接，实现一个连接多次请求

http/2.0 TCP 连接中上面的 http request 可以并发



原因 ： 防止因为网络延迟而造成服务端开启无用的连接

\- 超时重发

\- 规避网络传输中延时导致的一些服务器开销问题



HTTP 报文 ： 





# HTTP 特性

## 跨域请求

\- JSONP



\- Access-Controll-Allow-Origin:  是否允许跨域请求

​    |-- *:   不安全



**JSONP**

浏览器中的默认允许跨域：

<link>、<img>、<script> 标签上写路径加载允许跨域



JSONP 原理：

<script> 标签请求



发起请求出





proxy 实现





**跨域请求的限制**

保证服务端的安全

只允许方法：

\- GET、HEAD、POST





允许的 Content-Type：

\- text/plain

\- multipart/form-data

\- application/x-www-form-urlencoded





其他限制：

\- 请求头限制   ▲

`- XMLHttpRequestUpload对象均没有注册任何事件监听器 - 请求中没有使用ReadableStream对象`





**预请求**

允许的自定义头部

Access-Control-Allow-Header：   自定义头

Access-Control-Allow-Method:    方法允许

'Access-Contrl-Max-Age:     1000          在一定时间内允许，可省略预请求



Request Method: OPTIONS            // 预请求，告知浏览器允许的，获取服务端允许



# 应用

## 缓存

**（1） 可缓存性**

**public**:  http 任何经过的地方都可以缓存

**private**:   只有发起请求的浏览器可缓存

**no**-**cache**:    任何节点都不可

​    |-- 需要进行「验证」



**（2） 缓存过期机制**

**max-age**=<seconds>:    常用

s-maxage=<seconds>:    只有在代理服务器端设置，覆盖max-age

max-stale=<seconds>:  在期间内，max-age 过期，可使用过期缓存

​    |- brower 不使用

​    |- 只在发起端使用



对应的字段 ：

① Pragma ：ht tp 1.0 使用

② Expire ： 带时区的时间，格林日志时间

​       |-以服务端的时间进行控制的

​       |-Http/1.0 中，会优先处理 max-age 指令

③ Cache-control ：300     s单位的时间间隔

​       |-新增加的

​       |-解决客户端与服务端时间不一致情况

​       |-max-age=0   chrome browere自带

```
Cache-control: max-age=3232656 
```

```
Expires: Wed, 04 Jul 2018 07:26:36 GMT
```



（3） 重新验证：  少用

must-revalidate:  max-age 过期区server 获取

proxy-revalidate:    对代理服务器的





（4） 其他

no-store:   忽略缓存, max-age，当成新的请求

no-transform:    格式转换内容



**资源认证**

Last-Modified-

ETag

no-cache

（1） Last-Modified：

配合 If-Modified-Since 或 If-Unmodified-Since（少）

能否使用缓存



（2） Etag：  更加严格

\- 数据签名，任何修改导致签名不同

\- 配合 If-Match 或 If-Non-Match 使用

\- 对比资源的签名判断是否使用 cache



（3） no-cache

max-age=2000, no-cache:   仍然需要到 app server 验证





**相关问题**

@Q: 如何解决缓存不一致问题？

静态资源内容的 hash 码进行判断  ⇒  url 变化       ▲  常用解决方案







## Cookie

处理 HTTP 协议无状态特性，使用 Cookie 保存状态信息

每次在浏览器向同一服务器再次发送请求时被携带上，告知请求来自同一个用户，存在性能开销；

（1） 请求

```html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

（2） 响应

```html
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

（3） 分类

会话 Cookie：     浏览器关闭后自动删除，仅在会话期有效；

持久性 Cookie： 设置给定的过期时间(Expires) 或有效期 ( max-age ) 之后成为持久性的 Cookie；

```html
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```



**Cookie 的参数**

（1） domain

指定哪些主机可以接受 Cookie，不指定情况下默认为当前文档的主机( 不含子域名 )。

指定了域名，一般包含子域名； 

```

```



（2） Path

标识



（3） max-age

表示可以存在多长时间；



（4） HttpOnly

安全☆， CSRS 攻击，诱使用户访问攻击者网站，携带 Cookie

​    |-- 进制 JS 访问 cookie,  无法通过 document.cookie 访问



（5） secure

只在 https 的是否发送





**些意点**

（1） 多 Cookie 设置：

'Set-Cookie': ['id=12', 'name=abc']



（2） Cookie 的删除

需要创建一个一样的 Cookie 进行有效期的覆盖实现；



（3） Cookie 的持久化







##  Session

**Cookie 与 Session 的区别**







# 数据协商

**接受的类型**

Accept：

Accept-Encoding:    编码方式，主要用来限制 server 如何进行数据的压缩

​    |-- 压缩算法： gzip、deflate、br(new 压缩比高)

Accept-Language:  不同地区展示的语言，国际化时非必要

zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2       q: 权重

User-Agent:  浏览器确定， 移动端？PC 端？

​    |-- 客户端指明当前所用的浏览器





**返回的数据类型**

Content: 

- Content-Type: 从 Accept 中选择一种进行返回

- Content-Encoding:  对应

Accept-Language	

```
text/html
text/
application/*
application/json
X-Content-Type-Options: nosniff
```







**Content-Type**

form 表单：

enctype:

​    \- text/plain

​    \- multipart/data:  将文件与字符串拆分出来，作为二进制的数据传输

​    \- application/x-www-form-urlencoded





# CSP

Content-Security-Policy

安全

作用：

\- 限制资源获取

\- 报告资源获取越权





## GET 与 POST 比较

（） 与 GET 的比较

① 幂等性： GET 每次获得的结果一致，而 POST 通常用语修改的，对于爬虫扫描页面的 URL 之后进行调用，不能够使用 GET 方式来进行删除数组；

② 长度 | 编码： GET 通常长度被限制住，且编码只够是 ASCII，而 POST 长度不限制且编码支持通用形式；

③ 安全性： 安全的 HTTP 方法会改变服务器状态，为只读的，GET 安全，POST 可能改变状态；

传输数据的安全性，GET 明文传输不安全，POST 抓包分析不安全；

④ 可缓存性： GET 可以缓存， POST 大部分不可缓存；





对于安全性，有数据的安全性，也有对于服务端数据修改的安全性两种；





1.   GET跟POST的区别是什么？

（1）传参方式，GET放在url后面，post放在http的body，GET更不安全

（2）参数长度，浏览器对url后面的参数长度有限制，post也有限制，但是post要比get大得多。这是浏览器加的限制，跟Http协议无关

（3）GET的页面可以被缓存，POST的不可以

（4）  GET可以添加收藏，POST不可以

（5）GET可以后退刷新，POST刷新会重新提交数据。

（6）GET不能做文件上传，POST可以。

（7）以上都是表象，最根本的区别是语义上的区别：GET的语义是请求获取指定的资源。GET方法是安全、幂等、可缓存的（除非有 Cache-ControlHeader的约束）。POST的语义是根据请求报文对指定的资源做出处理，具体的处理方式视资源类型而不同。POST不安全，不幂等，（大部分实现）不可缓存。简单地说GET是获取数据，POST是修改数据。跟Restful还有点区别，Restful规范里面，GET是获取，POST是添加，PUT是修改，DELETE是删除。





## XMLHttpRequest

> 为客户端提供在客户端和服务器器之间传输数据的功能，提供通过一个 URL 来获取数据的简单方式，不会使整个页面刷新。
>
> 使得网页只更新一部分页面而不会打扰到用户，在 AJAX 中被大量使用。





# 基本结构

## 状态码

503： 







**输入 URL 做了哪些事**

1. DNS 解析
2. TCP 连接
3. 发送 HTTP 请求
4. 服务器处理请求并返回 HTTP 报文
5. 浏览器解析渲染页面
6. 连接结束