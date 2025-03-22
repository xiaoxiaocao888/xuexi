新建一个base.html文件
```html
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta charset="UTF-8">

        <title>{% block title %}{% endblock %}--知了博客</title>

        <link rel="stylesheet" href="{% static 'bootstrap5/bootstrap.min.css' %}">

        <script src="{% static 'bootstrap5/popper.min.js' %}"></script>

        <script src="{% static 'bootstrap5/bootstrap.min.js' %}"></script>

        <link rel="stylesheet" href="{% static 'css/base.css' %}">

        {% block head %}{% endblock %}

    </head>

    <body>

        <header class="p-3 text-bg-light border-bottom mb-3 ">

            <div class="container">

              <div class="d-flex flex-wrap align-items-center justify-content-center justify-content-lg-start">

                <a href="/" class="d-flex align-items-center mb-2 mb-lg-0 text-white text-decoration-none">

                  <img src="{% static 'images/logo.png' %}" alt="" height="40">

                </a>

                <ul class="nav col-12 col-lg-auto me-lg-auto mb-2 justify-content-center mb-md-0">

                  <li><a href="#" class="nav-link px-2 text-secondary">首页</a></li>

                  <li><a href="#" class="nav-link px-2 text-white">发布博客</a></li>

                </ul>

                <form class="col-12 col-lg-auto mb-3 mb-lg-0 me-lg-3" role="search">

                  <input type="search" class="form-control placeholder="搜索..." aria-label="Search">

                </form>

                <div class="text-end">

                  <button type="button" class="btn btn-outline-primary me-2">登录</button>

                  <button type="button" class="btn btn-primary">注册</button>

                </div>

              </div>

            </div>

          </header>

<main class="container bg-white p-3 rounded" >

  {% block main %}{% endblock %}

</main>

    </body>

</html>
```

其他模板继承eg:
```html
{% extends 'base.html' %}

{% block title %}发布博客{% endblock %}

{% block head %}

<link rel="stylesheet" href="{% static 'wangeditor/style.css' %}">

        <script src="{% static 'wangeditor/index.js' %}"></script>

        <script src="{% static 'js/pub_blog.js' %}"></script>

        <style>

            #editor—wrapper {

              border: 1px solid #ccc;

              z-index: 100; /* 按需定义 */

            }

            #toolbar-container {

              border-bottom: 1px solid #ccc;

            }

            #editor-container {

              height: 500px;

            }

          </style>

{% endblock %}

{% block main %}

<h1>发布博客</h1>

  <div class="mt-3">

    <form action="" method="POST">

        <div class="mb-3">

    <label class="form-label">标题</label>

    <input type="text" name="title" class="form-control">

  </div>

  <div class="mb-3">

    <label class="form-label" >分类</label>

    <select name="category" class="form-select">

        <option value="1">python</option>

        <option value="2">前端</option>

    </select>

  </div>

  <div class="mb-3">

    <label class="form-label">内容</label>

    <div id="editor—wrapper">

        <div id="toolbar-container"><!-- 工具栏 --></div>

        <div id="editor-container"><!-- 编辑器 --></div>

      </div>

  </div>

  <div class="mb-3 text-end">

    <button class="btn btn-primary">发布</button>

  

  </div>

</form>

</div>

{% endblock %}
```