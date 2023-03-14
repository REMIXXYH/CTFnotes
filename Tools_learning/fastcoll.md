#### 简述

fastcoll用于生成两个具有不同文件内容但是具有相同MD5值的文件



#### 使用

可以在shell.txt里写入一句话木马，-p的意思是以shell.txt的文件内容为前缀生成

```wiki
fastcoll.exe -p shell.txt -o ans.txt a.txt
```

