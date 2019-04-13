### HTTP的工作方式


http://www.runoob.com/http/http-tutorial.html


1. HTTP的工作模型：

以手机APP为例：


用户点击或界面自动触发联网需求---> Android代码调用拼装HTTP报文并发送请求到服务器——>服务器处理请求后发送响应报文给手机————> Android代码处理响应报文并作出相应处理。（存储、加工、显示数据到界面）

以浏览器为例：

用户输入地址后回车或者点击链接————> 浏览器拼装HTTP报文并发送请求给服务器 ---> 服务器处理请求后发送响应报文给浏览器 --> 浏览器解析响应报文并使用渲染引擎显示到界面。


2. URL 格式：

https://www.ethercap.com?token="asdfaweeav"

三部分：协议类型、服务器地址（和端口号）、路径（path）

3. 报文格式：

	请求报文：

	1. 请求行： get  /users http/1.1  (方法、path、http version)
	2. Headers

		Headers:
			host : 用于找到目标主机后确认主机域名和端口
			content-type
			content-length

	3. body


	响应报文 Response：

		1. 状态行:  HTTP/1.1 200  OK
		2. Headers
		3. body

====================================

#### Request 方法

get
	获取资源，没有body (幂等：反复调用多次时会得到相同的结果)
post
    增加或修改资源，有body
put 
    修改资源；有body(幂等：反复调用多次时会得到相同的结果)
delete
    删除资源，没有body(幂等：反复调用多次时会得到相同的结果)
 HEAD
    获取信息，断点下载，先获取下载的信息，是否支持断点续传；


=================================

关键内容：

status code: 200， 400， 500
	作用：对结果做出类型化的描述（获取成功、内容未找到）  

1xx 临时性消息 102 协议切换
2xx success
3xx 重定向   
4xx 客户端错误
5xx 服务端错误

==============================

header 首部:

作用： http的元数据： metadata

host: 服务器主机地址  host: api.ethercap.com 
     dns查询： domain name system (IP 地址在dns查询的过程中找到)

content-length: 内容长度；

content-type： 指定body的类型，

* text/html： 请求web页面是返回相应的类型，body中返回html文本
* applicaiont/x-www-form-urlencoded : 普通表单, 内容是给服务器看的  @FormUrlEncoded
* multipart/form-data： 多部分形式，一般用于传输包含二进制内容。@Multipart 
* application/json: json形式，用于web api的响应或者 post / put 请求。
* image/jpeg: 单文件
* application/zip 单文件


   boundary，分界线，相当于&的分割作用。

   【需要传文件的时候，才需要用multipart】

Retrofit的注解设计：

* text/html： 请求web页面是返回相应的类型，body中返回html文本
* applicaiont/x-www-form-urlencoded : 普通表单, 内容是给服务器看的  @FormUrlEncoded
* multipart/form-data： 多部分形式，一般用于传输包含二进制内容。@Multipart 
* application/json: json形式，用于web api的响应或者 post / put 请求。@Body
* image/jpeg: 单文件
* application/zip 单文件

Chunked Transfer Encoding: 
	目的：在服务端还未获取到完整内容时，更快对客户端做出响应，减少用户等待。

	Transfer: chunked.


=========================================

location: 指定重定向的URL。

User-Agent: 用户代理，即谁实际发送请求、接受响应的，例如：手机浏览器、某款手机APP。

Range/Accept-Range: 

Accept-Range: bytes: 响应报文中，表示服务器支持按字节来取范围数据（断点续传，多线程下载）。

Range:bytes = <start>-<end> 请求报文中出现，表示要取哪段数据

Content-Range: <start>-<end>/total 响应报文中出现，表示发送的是哪款数据。

其它Headers:

Accept: 客户端能接受的数据类型，如 text/html
Accept-Charset: 客户端接受的字符集，如 utf-8
Accept-Encoding: 客户端接受的压缩编码类型，如 gzip
Content-Encoding: 压缩类型，如gzip

Cache 和 Buffer的区别：

1. cache: 

	Cache-Control: no-cache, no-store, max-age
	Last-Modified: if-modified-Since

2. Buffer: 

======================================================

编码加密、Hash、序列化、字符集

=


密码学：


对称加密：原理（加密算法 + 解密算法）

> 通信双方使用同一个密钥，使用加密算法配合上密钥来加密，解密时使用加密过程的完全逆过程配合密钥来进行解密。

经典算法： DES， AES。

（密钥的泄露风险）

非对称加密：

原理： 使用公钥对数据进行加密得到密文，使用私钥对数据进行解密得到原数据。

每个人都有一对： 公钥、私钥。A、B 交换公钥，（公钥和私钥可以互解）


延伸： 数字签名。


经典算法： RSA， DSA（只能签名）



====

Base 64

将二进制数据转换成由64个字符组成的字符串的编码算法。

哪些字符： 字母，大小写 52个 和0到9 和 + /

什么是二进制数据？
> 只要不是文本数据，都是二进制数据。

用途：
> //TODO: 

Base58:

======

URL encoding: 


=====

压缩与解压缩：

压缩：把数据换一种方式更具有存储优势的编码算法进行编码。
解压缩：将压缩数据解码还愿成原来的形式，以方便使用。

目的：
> 减少数据占用的存储空间。

常见压缩算法： DEFLATE， JPEG， MP3

压缩属于编码啊？




=====

序列化：

 把数据对象（一般是内存中的，例如JVM中的对象）转换成字节序列的过程。

反序列化：
 
 把字节序列重新转换成内存中的对象

目的： 让内存中的对象可以被存储和传输。

======

Hash:

定义：把任意数据转换成指定大小范围的数据。

hash的实际用途: 
   数据完整性验证。
   快速查找（hashcode, hashmap）
   隐私保护

hash 不是 编码：

hash 不是加密。


签名的合法性校验：

====

字符集


 * ASCII
 * ISO-8859-1
 * Unicode: UTF-8 , UTF-16
 * GBK , GB2312, GB18030


====







































