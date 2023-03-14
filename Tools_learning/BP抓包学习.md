### Request

#### **X-Forwarded-For**

简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项

#### HTTP Referer

header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的

#### Authenticate(认证信息)

1. 客户端请求一个需要身份认证的页面，但是没有提供用户名和密码。这通常是用户在地址栏输入一个[URL](https://zh.wikipedia.org/wiki/URL)，或是打开了一个指向该页面的[链接](https://zh.wikipedia.org/wiki/超链接)。
2. 服务端响应一个401[应答码](https://zh.wikipedia.org/wiki/HTTP状态码)[[4\]](https://zh.wikipedia.org/wiki/HTTP基本认证#cite_note-section-11-4)，并提供一个认证域（英语：Access Authentication）头部字段为：`WWW-Authenticate`，该字段为要求客户端提供适配的资源。 `WWW-Authenticate: Basic realm="Secure Area"` 该例子，`Basic` 为验证的模式，`realm="Secure Area"`为保护域，用于与其他请求URI作区别。

​	3.接到应答后，客户端显示该认证域给用户并提示输入用户名和密码。此时用户可以选择确定或取消。

​	4.用户输入了用户名和密码后，客户端软件将对其进行处理，并在原先的请求上增加认证消息头（英语：Authorization）然后重新发送再次尝试。基本认证过程如下：

​			将用户名和密码拼接为`用户：密码`形式的字符串。

​			如果服务器`WWW-Authenticate`字段有指定编码，则将字符串编译成对应的编码（如：UTF-8）。

​			将字符串编码为base64。

​			拼接`Basic` ，放入`Authorization`头字段，就像这样：`Authorization Basic 字符串`。 示例：用户名：`Aladdin` ，密码：`OpenSesame` ，拼接后为`Aladdin:OpenSesame`，编码后`QWxhZGRpbjpPcGVuU2VzYW1l`，在HTTP头部里会是这样：`Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l`。 Base64编码并非加密算法，其无法保证安全与隐私，仅用于将用户名和密码中的不兼容的字符转换为均与[HTTP协议](https://zh.wikipedia.org/wiki/HTTP协议)兼容的字符集。

```

GET /private/index.html HTTP/1.0
Host: localhost
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```



### 爆破技巧

**1.可以在爆破字段前添加前缀**

![](D:\CTF\文档\BP抓包学习\字段前缀添加.jpg)

**2.可以对爆破字段编码**

![](D:\CTF\文档\BP抓包学习\对爆破字段编码.jpg)

