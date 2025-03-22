在templates文件夹建立html文件，在主文件里面加入代码
```
from flask import render_template
```
在对面的函数里面的返回为
```
return render_template("index.html") #对于的文件名
```

后端向前端传递数据，和fastapi是一样的
后端 参数=参数
```
@app.route("/blog/<blog_id>")
def blog_detail(blog_id):
    return render_template("blog_detail.html",blog_id=blog_id,username="李庆惠")
```
前端 用{{参数名}}
```
<p>您的用户名是:{{username}}</p>
<p>您访问的博客id是：{{blog_id}}</p>
```
![[Pasted image 20250207203738.png]]

类和字典的传递
后端
```
class User:
    def __init__(self,username,email):
        self.username=username
        self.email=email
@app.route('/')
def hello_world():
    user=User(username="丽丽",email="xx@qq.com")
    person={
        "username":"小小",
        "email":"xxxxxxx@qq.com"
    }
    return render_template("index.html",user=user,person=person)
```
前端
```
<div>{{user.username}}</div>/{{user.email}}
        <div>{{person['username']}}</div>/{{person['email']}}
        <div>{{person.username}}</div>/{{person.email}}
```

##### 过滤器的使用
返回后端传递过来的参数长度，在前端
```
{{参数|length}}
```
其他内置过滤器可以去这个官方学习
```
https://jinja.palletsprojects.com/en/3.0.x/templates/
```
##### 自定义过滤器
函数的第一个参数是需要被处理的值。下面通过app.add_template_filter将函数注册成过滤器，第一个参数的函数名，第二个参数是过滤器名
```
def datetime_format(value,format="%Y年%m月%m日 %H:%M"):
    return value.strftime(format)
app.add_template_filter(datetime_format,"dformat")
```
使用自定义过滤器{{参数|过滤器名}}

前端页面对参数进行限制，使用控制语句
```
{% if 参数的限制条件 %}
   <div></div>
{% elif 参数的限制 %}
   <div></div>
{% endif %}
```

```
{%for book in books %}

        <div>图书名称：{{book.name}} ，图书名称：{{book.author}}</div>

        {% endfor %}
```
![[Pasted image 20250207215001.png]]

使用jinja2模板加载静态文件，第一个参数一定是static,第二个参数就是文件路径
```
<link rel="stylesheet" href="{{url_for('static',filename='css/style.css')}}">
```

##### 模板的继承
父模板base.html
```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>{% block title %} {% endblock %}</title>
    </head>
    <body>
       <ul>
        <li><a href="#">首页</a></li>
        <li><a href="#">新闻</a></li>
       </ul>
       {% block body %}
       {% endblock %}
       <footer>这是底部的标签</footer>
    </body>
</html>
```

子模板child1.html
```
{% extends "base.html" %}
{% block title %}
我是子模板的标题
{% endblock %}

{% block body %}
我是子模板的body
{%endblock %}
```
![[Pasted image 20250207222602.png]]

##### 前端引入css,js
Flask 框架约定，所有的静态文件（如 CSS 文件、JavaScript 文件、图片等）都应该存放在项目根目录下名为 `static` 的文件夹中。当你使用 url_for函数 时，Flask 会根据这个 `static` 端点生成一个指向静态文件的 URL。
```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" href="{{url_for('static',filename='css/style.css')}}">
        <script src="{{url_for('static',filename='js/my.js')}}"></script>
    </head>
    <body>
        <img src="{{url_for('static',filename='images/1.jpg')}}"
    </body>
</html>
```
