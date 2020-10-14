[TOC]

#### HTTP定义

##### HTTP

超文本传输协议

##### HTTP的工作方式

- 浏览器 — 用户输入地址后回车或点击链接—> 浏览器拼接HTTP报文并发送请求给服务器 —>服务器处理请求后发送响应报文给浏览器 —> 浏览器解析响应报文并使用渲染引擎显示到界面
- 手机App — 用户点击或界面自动触发联网需求 —> Android代码调用拼接HTTP报文发送请求到服务器 —> 服务器处理请求后发送响应报文给手机 —> Android代码处理响应报文并作出响应处理

#### URL和HTTP报文

##### URL格式

- 三部分 : 协议类型,服务器地址(和端口号),路径(Path)

- 协议类型 : //服务器地址[:端口号]路径

  `http://hencoder.com/users?gender=male`

#### 报文格式

- 请求报文

  ```
  请求行 POST /users Http/1.1 (method,path,Http version)
  Headers -- Host:api.github.com,
  					 Content-Type:text/plain,
  					 Content-Length:243}
  Body 
  ```

- 响应报文

  ```
  状态行 HTTP/1.1 200 OK (HTTP version , statuc code , staus message)
  Headers -- content-type:application/json;charset=utf-8,
  					 cache-control:public,max-age=60,s-maxage-60,
  					 vary:Accept,Accept-Encoding,
  					 etag:W/"",
  					 content-encoding:gzip
  Body	{}
  ```

#### http四种请求方式的作用

- GET
  - 获取资源,没有Body
- POST
  - 增加或修改资源:有Body
- PUT
  - 修改资源:有Body(只用来修改)
- DELETE
  - 删除资源:没有Body

- HEAD
  - 和GET几乎是一样的,区别是HEAD服务器返回响应报文时没有Body

注:PUT、GET和DELETE都是幂等的,这个请求,请求一次和多次结果是一样的

#### Status Code状态吗

- 1xx : 临时性消息.如100(继续发送),101(正在切换协议(支持Http2))
- 2xx : 成功.最典型的是200(OK),201(创建成功)
- 3xx : 重定向,如301(永久移动),302(暂时移动),304(内容未改变)
- 4xx : 客户端错误如400(客户端请求错误),401(认证失败),403(被禁止),404(找不到)
- 5xx : 服务器错误,如500(服务器内部错误)

#### Header首部

HTTP消息的metadata

##### Host

目标主机.注意:不是在网络上用于寻址的,而是在目标服务器上用于定位子服务器的

注:寻址是使用域名使用DNS实现

##### Content-Type

指定Body类型

- text/html : 请求Web页面返回响应的类型,Body中返回html文本(html文件)
- x-www-form-urlencoded : Web页面纯文本表单的提交方式 Body格式(name='yangdxg'&'nickName=yangdong')
- multitype/form-data : Web页面含有二进制文件时的提交方式
- application/json : 单项内容(文本或非文本都可以),用于Web Api的响应或者Post/Put的请求(请求中提交Json,响应中返回Json)
- image/jpeg , application/zip 请求中提交二进制文件,响应中返回二进制文件

##### Content-Length

指定Body的长度

##### Transfer:chunked(分块传输编码) 

当响应发起时,内容长度还没能确定的情况下,和Content-Length不同时使用,用途是尽早给出响应,减少用户等待`Transfer-Encoding: chunked`(将已有的数据先传回去,剩余数据进行计算,表示Body长度无法确定,Content-Length不能使用,最后传输0表示内容结束)

##### Location

指定重定向的目标URL

##### User-Agent

用户代理,即是谁实际发送请求,接受响应的

##### Range/Accept-Range

按范围取数据

作用:断点续传,多线程下载

- `Accept-Range:bytes`响应报文中出现,表示服务器支持按字节来取范围数据
- `Range:bytes=<start>-<end>`请求报文中出现,表示要取哪段数据
- `Content-Range:<start>-<end>/total`响应报文中出现,表示发送的是哪段数据

##### 其它Headers

- Accept:客户端能接受的数据类型,如text/html
- Accept-Charset:客户端接受的字符集,如utf-8
- Accept-Encoding:客户端接受的压缩编码类型,如gzip
- Content-Encoding:压缩类型,如gzip

#### Cache

作用:在客户端或中间网络节点缓存数据,降低从服务器取数据的频率,以提高网络性能

- Cache和Buffer的区别
  - Cache缓存,Buffer缓冲
  - Cache缓存是一会还有可能会用
  - Buffer缓存是针对工作流的,上游多生产一些等待下游用

- Canche-Control:
  - no-cache服务器告诉客户端可以缓存,当需要再次使用该资源时,需要询问服务器资源是否过期
  - no-store不缓存
  - max-age定义一个失效日期,再失效之前一直可以用
  - private/public服务器告诉中间节点是否要帮助缓存这些消息

- Last-Modified询问服务器时返回最近什么时候改变的
  - if-Modified-Since是否在什么时候之前改动过

- Etag给资源帖了一个标签,
  - if-None-Match

#### REST

REST HTTP即正确使用HTTP.

- 使用资源的格式来定义URL
- 规范的使用method来定义网络请求操作
- 规范的使用status code来表示响应状态
- 其它符合HTTP规范的设计准则

