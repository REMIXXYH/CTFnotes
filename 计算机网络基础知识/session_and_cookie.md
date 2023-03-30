## session

​		`Session`一般称为“会话控制“，**是一种在服务端和客户端之间保持状态的方案**。一旦开启了 `session` 会话，便可以在网站的任何页面使用或保持这个会话，从而让访问者与网站之间建立了一种“对话”机制。



#### session的创建

**情况一：用户从未在该网站活动过**

​		即用法发送的请求包里没有sessionId，这种情况下，服务端会为客户创建一个session并且生成一个与此session相关联的sessionId，这个**sessionId将在本次响应中返回给客户端保存**。

![](D:\0-CTF\Documents\计算机网络基础知识\images\session_1.jpg)



**情况二：该用户曾在该网站活动过**

​		这个时候用户在发送数据包时会携带sessionId，服务端会检查该Id是否存在过，如果检索不到，则再次新建一个sessionId，这种情况可能出现在服务端已经删除了该用户对应的session对象，但用户人为地在请求的URL后面附加上一个JSESSION的参数)。



**存储方式**

​		返回的sessionId一般以cookie的形式存放在浏览器。

​		当禁用cookie时，可以通过URL重写的形式人为添加sessionId，`;Jsessionid=ASJLKJDSL234JFL967DSKJFL87DH`