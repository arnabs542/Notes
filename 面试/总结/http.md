## 1.**键入网址再按下回车的全过程**
**http方面**
- 浏览器从地址栏的输入中获得服务器的 IP 地址和端口号；
- 浏览器用 TCP 的三次握手与服务器建立连接；
- 浏览器向服务器发送拼好的报文；
- 服务器收到报文后处理请求，同样拼好报文再发给浏览器；
- 浏览器解析报文，渲染输出页面。


## 2.**输入一个url发生了什么？**

- 浏览器从地址栏的输入中获得服务器的 IP 地址和端口号；
- 浏览器用 TCP 的三次握手与服务器建立连接；
- 浏览器向服务器发送拼好的报文；
- 服务器收到报文后处理请求，同样拼好报文再发给浏览器；
- 浏览器解析报文，渲染输出页面。

## 3.**解释一下在浏览器里点击页面链接后发生了哪些事情吗？**
- 如果连接地址是域名开头的，浏览器会开始DNS解析动作。解析的顺序依次为: 浏览器缓存 -> 操作系统缓存 -> 本机的Host文件 ->  “野生的DNS服务器” -> 核心服务器(根DNS、顶级DNS、权威DNS)。
- 将域名解析为正确的ip地址以后，通过三次握手与服务器建立tcp/ip的连接。
- 浏览器发送拼接好的请求报文。
- 服务器接收到处理请求，返回响应报文
- 浏览器接收到响应报文，开始解析html文档。在这个过程中又会发起一些http请求，进行图片、css、js等静态资源的获取，以及ajax请求获取数据。
- 同时，浏览器引擎开始绘制DOM视图，执行js脚本，完成页面的初始化直到所有代码执行完毕。


## 4.**如果是一个不存在的域名，那么浏览器的工作流程会是怎么样的呢？**
- 当我们输入一个不存在的域名，依然会走上题的DNS解析顺序。当请求DNS服务器域名解析时，发现没有找到对应的ip地址，会导致ip地址解析失败。
- 因为ip地址解析失败，会导致浏览器建立连接时间过长，最终建立连接失败，浏览器停止建立连接动作。

## 5.http协议中空白行的意义
请求行 + 头部信息 + 空白行 + body。
空白行的意义为了分割header和body，因为http是纯文本的协议。

## 6.**http的标准方法**
- 请求方法是客户端发出的、要求服务器执行的、对资源的一种操作；
- 请求方法是对服务器的“指示”，真正应如何处理由服务器来决定；
- 最常用的请求方法是 GET 和 POST，分别是获取数据和发送数据；
- HEAD 方法是轻量级的 GET，用来获取资源的元信息；
- PUT 基本上是 POST 的同义词，多用于更新数据；
- “安全”与“幂等”是描述请求方法的两个重要属性，具有理论指导意义，可以帮助我们设计系统。

## 7.**URI**
- URI 是用来唯一标记服务器上资源的一个字符串，通常也称为 URL；
- URI 通常由 scheme、host:port、path 和 query 四个部分组成，有的可以省略；scheme 叫“方案名”或者“协议名”，表示资源应该使用哪种协议来访问“host:port”表示资源所在的主机名和端口号；
- path 标记资源所在的位置；
- query 表示对资源附加的额外要求；
- 在 URI 里对“@&/”等特殊字符和汉字必须要做编码，否则服务器收到 HTTP 报文后会无法正确处理。
  
## 8.**HTTP 协议允许在在请求行里使用完整的 URI，但为什么浏览器没有这么做呢？**
不需要重复写，在head里面有的

## 9.URI 的查询参数和头字段很相似，都是 key-value 形式，都可以任意自定义，那么它们在使用时该如何区别呢？
字段是针对这次请求的，query是针对访问的资源