### 2023

#### 2.28

**攻防世界**

**1.unseping：**

php反序列化，命令注入的绕过`l\s`，`$(printf${IFS}"8进制编码")`

**BUUCTF**

**2.极客大挑战HardSQL：**

SQL注入，报错注入，括号拼接绕过空格过滤，like字段绕过=过滤，通过左右显示来突破报错注入的回显长度限制

#### 3.1

**BUUCTF**

**1.easy_md5：**

sql中的md5函数转义漏洞，php中的md5弱类型比较漏洞，通过0e开头的hash绕过和数组不能解析漏洞绕过

**2.Nizhuansiwei:**

php伪协议读取文件，filter结合file_get_contents函数读取源码，data://用于将字符串流认作文件使用，php简单反序列构造

**3.[MRCTF2020]Ez_bypass：**

php的MD5数组弱类型绕过，数字与字符串比较弱类型绕过

**4.[SUCTF 2019]CheckIn:**

.user.ini文件上传，配合.gif和利用js代码对<?进行bypass，最后使用蚁剑getshell

#### 3.2

**BUUCTF**

**1.[GXYCTF2019]BabySQli**

对后端代码的猜测，账号密码分两次查询，通过sqlmap查出来后发现密码经过md5加密，那么我们可以MD5对数组不能解析来绕过

**2.[GXYCTF2019]BabyUpload**

文件上传中.htaccess文件的利用

**CTFshow**

**3.web4**

Linux下的web访问日志文件，通过修改header，实现文件包含下的getshell

#### 3.4

**1.CTFshow-web8**

bool盲注的脚本编写

#### 3.5

**2.CTFshow-web9**

SQLi，对md5($str,true)的理解，原始二进制串

**3.CTFshow-web12**

使用UA注入getshell语句到nginx的访问日志，然后使用文件包含，并使用蚁剑连接，使用了disable_function的插件

```url
http://9b1ed8c3-fd3a-4ed4-a862-8f510b423d20.challenge.ctf.show/index.php?cmd=include('/var/log/nginx/access.log')
```

也可以使用另一种payload`?cmd=print_r(glob("*"));`

#### 3.6

**1.CTFshow-红包题2**

无数字字母webshell，PHP文件上传保存临时文件利用文件包含getshell

#### 3.8

**1.CTF-show-web13**

.user.ini文件利用，文件上传，getshell

**2.CTF-show-web14**

switch语句的break;逻辑漏洞，简单的union型sql注入，Mysql利用load_file函数读取文件

**3.BUUCTF-网鼎杯2020-Nmap**

Nmap基本命令`-oG`使用，php函数escapeshellarg和escapeshellcmd联合造成的漏洞

#### 3.9

**1.CTF-show菜狗杯-web签到**

php代码利用，`eval($_REQUEST[$_GET[$_POST[$_COOKIE['CTFshow-QQ群:']]]][6][0][7][5][8][0][9][4][4]);`，这是层层嵌套，同时传cookie`CTFshow-QQ群:=a`，POST` a=b`，GET`b=c`，`c=d$d[6][0][7][5][8][0][9][4][4]=system('ls')`

**2.极客大挑战-RCEME**

无数字字母shell编写，利用取反绕过，中国蚁剑插件绕过disable_function

#### 3.14

**1.CTFshow红包题第六弹**

并发编程，MD5hash

**2.CTFshow萌新专属红包题**

弱口令，签到题

#### 3.15

**1.CTFshow红包题第七弹**

Git泄露，新的Git泄露工具Git_extract获取源码，

#### 3.16

**1.ctfshow-给她**

sprintf函数解析漏洞，将无法解析占位符替换为空，从而实现单引号逃逸

**2.ctfshow-萌新赛签到题**

简单的命令注入，通过||实现，payload：||ls||,||cat flag|

**3.ctfshow-萌新赛假赛生**

正则未过滤空格，

#### 3.18

**1.ctfshow-萌新记忆**

布尔盲注

**2.ctfshow-单身杯-web签到**

data://协议配合一个php字符反转函数

