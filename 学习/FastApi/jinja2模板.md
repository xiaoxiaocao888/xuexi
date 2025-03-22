模板在python的web开发中广泛使用，它能够有效的将业务逻辑和页面逻辑分开，所以一般不会在main文件中使用info[0]这样的语句
模板就是一个其中包含占位变量表示动态部分的文件，模板文件在经过动态赋值后，返回给用户。
jinja2是Flask作者开发的一个模板系统，可作为第三方库，起初是仿django模板的一个模板引擎。
变量取值用{{}}
控制结构{% %}

代码 ，在根目录下新建templates文件，再创建index.html文件
```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <p>用户名：{{user}}</p
        <p>年龄：{{age}}</p>
        <p><1>:{{books}}</p>
        <p><1>:{{books.0}}</p>
            <p><2>:{{books.1}}</p>
                <p><3>:{{books.2}}</p>
                    <p><4>:{{books.3}}</p>
        <ul><li>{{books.0}}</li></ul>
        <p>姓名：{{info.name}}</p>
        <p>年龄：{{info.age}}</p>
        <p>性别：{{info.gender}}</p>
    </body>
</html>
```

代码 main文件
```
from fastapi import FastAPI, Request
import uvicorn
from fastapi.templating import Jinja2Templates

app=FastAPI()
templates=Jinja2Templates(directory="templates")
@app.get("/index")
def index(request:Request):
    name="root"
    age=32
    books=["精品","聊斋","建筑","果实"]
    info={"name":"rain","age":33,"gender":"male"}
    return templates.TemplateResponse(
        "index.html",#模板文件
        {
            "request":request,
            "user":name,
            "age":age,
            "books":books,
            "info":info
        },#context上下文对象，一个字典
    )
if __name__=='__main__':
    uvicorn.run("main:app",port=8090,reload=True)
```
![[Pasted image 20250122121158.png]]
响应数据渲染到html里面了
![[Pasted image 20250122121144.png]]

##### jinja2 过滤器
把业务逻辑层的数据在模板语法层做一个更好的数据展示，变量可以通过“过滤器”进行修改，过滤器可以理解为是jinja2里面的内置函数和字符串处理函数，常用的过滤器有
capitalize       把值的首字母转换成大写，其他字母转换为小写
lower              把值转换成小写
upper             把值转换成大写
title                 把值中的每个单词的首字母都转换成大写
trim                 把值的首尾空格去掉
striptags          渲染之前把值中所有的HTML标签都删掉
join                  拼接多个值为字符串
round               默认对数字进行四舍五入，也可以用参数进行控制
safe                   渲染时值不转义

代码index.html
```
<p>字母：{{str1}}</p>
        <p>字母大写：{{str1|upper}}</p>
        <p>字母首字母大写，其他小写：{{str1|capitalize}}</p>
        <p>字母：{{str1|upper}}</p>
```
在main.py也要加上str1="abc"等代码
![[Pasted image 20250122124047.png]]

##### 控制语句
代码在index.html里面加
```
{% if age>18 %}
           <ul>
            <li>日韩</li>
            <li>欧美</li>
           </ul>
        {% endif %}
```

for循环
```
<ul>
            {% for book in books %}
            <li>{{book}}</li>
            {% endfor %}
        </ul>
```
if和for是可以相互嵌套的，jinja2里面是没有while语句的


