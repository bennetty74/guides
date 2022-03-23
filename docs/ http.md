# Http

此文主要讲解HTTP的相关知识。

HTTP全称为超文本传输协议(***Hypertext Transfer Protocol***)，是处在应用层的协议。起初是用来获取网络上的相关媒体资源的。

## URI和资源

为了标识网络上的资源，出现了URI。

URI全称为统一资源标识(Uniform Resource Identifier)，是用来标识网络资源的一种规范。

URI由很多部分组成：协议、主机、端口、路径、查询等。

其中URL(Uniform Resource Locator，统一资源定位符)与URI几乎相同，在http中最为常用，如我们浏览器地址栏的地址`https://bugkit.cn`。

**以HTTP为例：**

`http://`是协议，类似的还有`ftp`，`ssh`等等。

`ww.example.com`为主机

`80`为端口

`/path/to/myfile.html` 为路径

`?key1=value1&key2=value2`为查询条件

`#SomewhereInTheDocument` 为文件章节，英文称呼`fragment`

```
http://www.example.com:80/path/to/myfile.html
?key1=value1&key2=value2#SomewhereInTheDocument
```



## HTTP消息格式



HTTP是基于请求-响应模式下的一种应答协议，对应规范也分为两种，分别是Request和Response。



### Request

Request中文意思即为请求。请求分为请求行、请求头、空行和请求踢四个部分。

一个典型的请求报文如下：

```
POST /api/v2/za/logs/batch HTTP/1.1
Host: zhihu-web-analytics.zhihu.com
Connection: keep-alive
Content-Length: 457
X-ZA-ClientID: cfb262c0-19e2-4c90-b9f0-901f2801e45c
Content-Encoding: gzip
Content-Type: application/json; charset=UTF-8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) 

{
    "username":"username"
}
```

#### 请求行

上述内容的第一行为请求行，包含请求方法(POST)、请求路径(/api/v2/za/logs/batch)、HTTP版本(HTTP/1.1)。

请求方法除了POST请求，还包含GET、DELETE、PUT、OPTION等请求方法。



#### 请求头

第二行到第八行是请求头，请求头是可选的，主要用来描述请求的相关格式或者要求。

请求头每行以`key: value`的形式进行描述。

如Host是请求的主机域名，Content-Length向服务器指明请求体的内容长度，User-Agent表明请求的客户端为Mozilla/5.0。



> 请求头对大小写不敏感。



#### 空行

空行没有特殊的含义，主要作用是用来分割请求头和请求体。



#### 请求体

空行以下的内容为请求体。请求体是可选的，如GET请求就没有请求体，其查询参数是放在请求行的请求路径下的。

```
GET /test.html?query=alibaba HTTP/1.1
```





请求体主要是为了处理长请求内容而出现的，比如上传文件、提交表单信息。



### Response



Response中文意思为响应，即对请求作出的回答。和Request相比，响应同样分为相应行、响应头、空行和响应体四部分。



如下是一段典型的响应。

```
HTTP/1.1 200 OK
Date: Wed, 23 Mar 2022 12:00:09 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 0
Connection: keep-alive
Server: openresty
Access-Control-Allow-Origin: *
X-Backend-Response: 0.003 

<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    Hello World
</body>
</html>
```



#### 响应行

第一行为响应行，包含三部分：HTTP版本(HTTP/1.1)、状态码(200)和状态文本(OK)。



### 响应头

响应头的格式和请求头的格式基本一样的，这里不再赘述。



#### 空行

作分割响应头和响应体用。



#### 响应体

返回客户端的相应数据，如上就是返回客户端一段html代码。如果客户端是浏览器，浏览器就会根据相应头的content-type和响应体将响应体渲染到浏览器中。



# HTTP连接管理

什么是HTTP连接？

HTTP连接指的是在应用层下层的TCP连接或者UDP，通常是TCP。TCP为HTTP协议提供了数据传输的功能，使得我们的数据可以完整无误的到达目标程序(如浏览器)。

HTTP连接管理通常指的是TCP的连接管理。HTTP要传输数据，首先需要建立TCP连接，而TCP连接需要经过三次握手，握手成功后才会开始真正的HTTP的数据传输。数据传输结束后我们需要根据情况是否断掉传输完成的TCP连接。

这样的整个过程称为HTTP连接管理。

**HTTP1.0时，默认是短链接**。意思是说每次发送Request的时候，需要通过TCP三次握手建立连接，然乎传输数据，最后四次挥手断掉连接。

与短连接对应的肯定有长连接，指的是数据传输完成后不会马上断掉TCP连接。

如何设置为长链接？

如果还记得之前提过的请求头的话，可以很明显地知道通过请求头控制啦。

```
Connection: keep-alive
```

如上请求头就告诉服务器要保持长链接了。



HTTP1.1默认是长链接。

可以通过如下命令关闭长链接。

```
Connection: close 
```

 

### 长链接VS短链接

长链接适用于请求密集，保持长链接有利于减少TCP握手和挥手的时间，从而提供高性能。

短链接适用于资源闲置的情况，这样TCP连接空闲时可以保证服务器不会保证负载过高。





> 未完待续。。。
> 
> 最后更新日期：2022.03.23
