### gopher协议

#### 简介

   gopher协议支持发出GET、POST请求：可以先拦截get请求包和post请求包，再构造成符合gopher协议的请求。gopher协议是ssrf利用中一个最强大的协议（俗称万能协议）。可以攻击内网的 FTP、Telnet、Redis、Memcache，也可以进行 GET、POST 请求，还可以攻击内网未授权MySQL。

#### 构造

gopher格式：

`gopher://IP:port/_{TCP/IP数据流}`



##### gopher实现http请求

分为3步：

```
1、构造HTTP数据包
2、URL编码、替换回车换行为%0d%0a
3、发送gopher协议
```

==编码注意事项==

技巧：使用记事本的替换功能对%0a进行替换

```
1、问号（？）需要转码为URL编码，也就是%3f
2、回车换行要变为%0d%0a,但如果直接用工具转，可能只会有%0a
3、在HTTP包的最后要加%0d%0a，代表消息结束
```

##### 结合curl实现ssrf

cURL是一个利用URL语法在命令行下工作的文件传输工具。它支持文件上传和下载，所以是综合传输工具。cURL还包含了用于程序开发的libcurl。

curl格式：`curl protocol://address:port/url?args`

==注意在浏览器中传入的需要两次url编码，因为浏览器会解码一次==

php中的curl支持Gopher协议，所以有ssrf漏洞

##### CTFHUB上传文件Gopher题解

​        首先使用file协议暴露flag.php源码，发现只要上传文件就可以得到flag。所以本题考察通过内网访问上传文件，采用Gopher协议，我们直接访问flag.php发现需要通过127.0.0.1才行，所以访问`?url=127.0.0.1/flag.php`，发现有一个文件上传的选择文件按钮，但是没有上传，所以我们修改网页前端，加上按钮，提交后，用BP截下，然后修改一下，使得有文件上传，复制请求报文，进行两次编码，注意**Gopher协议里的换行符需替换**，在浏览器中输入`?url=gopher://127.0.0.1：80/_[url编码后的报文]`，访问后即可得到flag



#### gopherus工具使用

##### 攻击mysql

只在数据库没有设置密码的时候可以使用

首先输入用户名：`root`

然后输入要执行的sql语句：`select '<?php @eval()?>' INTO OUTFILE '/var/www/html/f.php'`

然后即可生成payload
