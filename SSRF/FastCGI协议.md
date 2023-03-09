### FastCGI协议

详见https://blog.csdn.net/mysteryflower/article/details/94386461

#### 简介

​       **FastCGI协议和HTTP协议一样是通信协议，不过，http是浏览器（也就是客户端）和服务器中间件之间通信的协议，而FastCGI协议是服务器中间件和某种语言编写的正在运行的后端程序间的通信协议**。

![](D:\CTF\文档\SSRF\image\FastCGI_1.jpg)

​       **在网站架构中，Web Server（如Nginx）只是内容的分发者。当客户端请求的是index.php，根据配置文件Web Server辨别不是静态文件，此时就需要去找 PHP解析器来处理。**

​       **FPM其实是一个fastcgi协议解析器，Nginx等服务器中间件将用户请求按照fastcgi的规则打包好通过TCP传给谁？其实就是传给FPM。FPM按照fastcgi的协议将TCP流解析成真正的数据。**

![](D:\CTF\文档\SSRF\image\FastCGI_2.jpg)



#### FastCGI协议漏洞详解

​        Fastcgi协议**由多个record组成**，**record也有header和body一说**，服务器中间件将这二者按照fastcgi的规则封装好发送给语言后端，语言后端解码以后拿到具体数据，进行指定操作，并将结果再按照该协议封装好后返回给服务器中间件。

##### 协议框架简介

```c++
typedef struct {
  /* Header */
  unsigned char version; // 版本
  unsigned char type; // 本次record的类型
  unsigned char requestIdB1; // 本次record对应的请求id
  unsigned char requestIdB0;
  unsigned char contentLengthB1; // body体的大小
  unsigned char contentLengthB0;
  unsigned char paddingLength; // 额外块大小
  unsigned char reserved; 
 
  /* Body */
  unsigned char contentData[contentLength];
  unsigned char paddingData[paddingLength];
} FCGI_Record;
```

**1.  和HTTP头不同，record的头固定8个字节，body是由头中的contentLength指定**

**2.**  因为fastcgi一个record的大小是有限的，作用也是单一的，所以我们需要在一个TCP流里传输多个record。通过`type`来标志每个record的作用，用`requestId`作为同一次请求的id。也就是说，**每次请求，会有多个record，他们的`requestId`是相同的**。**当type为4的时候，解析为键值对，就是在传递环境变量**

![](D:\CTF\文档\SSRF\image\FastCGI_4.jpg)



##### FPM

FPM是一个fastcgi协议解析器，按照fastcgi的协议将TCP流解析成真正的数据。

一部分经过中间件解析后的键值对示例：

```c
{
    'GATEWAY_INTERFACE': 'FastCGI/1.0',
    'REQUEST_METHOD': 'GET',
    'SCRIPT_FILENAME': '/var/www/html/index.php',
    'SCRIPT_NAME': '/index.php',
    'QUERY_STRING': '?a=1&b=2',
    'REQUEST_URI': '/index.php?a=1&b=2',
    'DOCUMENT_ROOT': '/var/www/html',
    'SERVER_SOFTWARE': 'php/fcgiclient',
    'REMOTE_ADDR': '127.0.0.1',
    'REMOTE_PORT': '12345',
    'SERVER_ADDR': '127.0.0.1',
    'SERVER_PORT': '80',
    'SERVER_NAME': "localhost",
    'SERVER_PROTOCOL': 'HTTP/1.1'
}
```

PHP-FPM拿到fastcgi的数据包后，进行解析，得到上述这些环境变量。然后，执行`SCRIPT_FILENAME`的值指向的PHP文件，也就是`/var/www/html/index.php`。

##### 漏洞的形成

​		PHP-FPM默认监听9000端口，如果这个端口暴露在公网，则我们可以自己构造fastcgi协议，和fpm进行通信。本来我们可以执行任意文件，但是在FPM某个版本之后，增加了`security.limit_extensions`选项，限定了只有某些后缀的文件允许被fpm执行，默认是`.php`，所以现在**要使用这个漏洞，我们得先知道服务器上的一个php文件**，假设我们爆破不出来目标环境的web目录，我们可以找找默认源安装后可能存在的php文件，比如`/usr/local/lib/php/PEAR.php`。

​		现在我们已经有可以执行代码的思路了，**但是仅仅这样我们只能执行服务器上已有的代码，不能任意执行**，所以我们还得想办法上传自己的代码，**PHP.INI中有两个有趣的配置项，`auto_prepend_file`和`auto_append_file`。**

`auto_prepend_file`是告诉PHP，在执行目标文件之前，先包含`auto_prepend_file`中指定的文件；`auto_append_file`是告诉PHP，在执行完成目标文件后，包含`auto_append_file`指向的文件。

​		**假设我们设置`auto_prepend_file`为`php://input`，那么就等于在执行任何php文件前都要包含一遍POST的内容。所以，我们只需要把待执行的代码放在Body中，他们就能被执行了。（当然，还需要开启远程文件包含选项`allow_url_include`）**。设置`auto_prepend_file`的值又涉及到PHP-FPM的两个环境变量，`PHP_VALUE`和`PHP_ADMIN_VALUE`。这两个环境变量就是用来设置PHP配置项的，`PHP_VALUE`可以设置模式为`PHP_INI_USER`和`PHP_INI_ALL`的选项，`PHP_ADMIN_VALUE`可以设置所有选项。（`disable_functions`除外，这个选项是PHP加载的时候就确定了，在范围内的函数直接不会被加载到PHP上下文中），所以我们在传入环境变量是加入

```c++
'PHP_VALUE': 'auto_prepend_file = php://input',
'PHP_ADMIN_VALUE': 'allow_url_include = On'
```

即可。

以上就是漏洞原理，以此大师傅写下fpm.py脚本

#### Gopherus工具ssrf攻击FastCGI

Kali环境中，使用shell，`python2 gopherus.py --expolit fastcgi`

第一行键入已知目标服务器上的.php文件，第二行键入shell语句，直接生成Gopher语句，在进行二次url编码，找到攻击窗口攻击。

![](D:\CTF\文档\SSRF\image\FastCGI_3.jpg)