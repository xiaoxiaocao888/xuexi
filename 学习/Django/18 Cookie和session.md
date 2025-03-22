1.cookie：在网站中，http请求是无状态的，也就是说即使之前已经访问过服务器，第二层请求服务器依然不知道当前请求是谁。cookie可以解决这个问题，在第一次登录后服务器返回一些数据（cookie）给浏览器，然后浏览器保存在本地，而当用户发送第二次请求的时候，就会自动地把上次请求存储的cookie数据自动的携带给服务器，服务器通过浏览器携带的数据就能判断当前的用户是哪一个。cookie存储的数据量有限，不同的浏览器有不同的存储大小，但是一般不超过4KB。因此cookie只能存储一些小量的数据

2.session：session是一个思路、一个概念、一个服务器存储授权信息的解决方案，不同的服务器，不同的框架，不同的语言有不同的实现。session是为了解决cookie存储数据不安全问题的

3.cookie和session:要是存储在服务端，那就是通过cookie存储一个sessionid，然后具体的数据则是保存在session，服务器根据sessionid在session库中获取用户的session数据来知道该用户到底是谁，以及之前保存的一些状态信息。这种专业术语叫做server side session。django把session信息默认存储到数据库中，当然也可以存储到其他地方，如缓存、文件系统等。存储在服务器的数据会更加安全，不容易被窃取，但是会占用服务器的资源
将session数据加密，然后存储在cookie中，这种专业术语叫做client side session。flask框架默认采用的就是这种方式，但是也可以替换成其他形式

##### 在django中操作cookie
1.设置cookie
设置cookie是设置值给浏览器的。因此我们需要通过response的对象来设置，设置cookie可以通过response.set_cookie来设置，这个方法的相关参数如下
- key/value:这个cookie的名字/值
- max_age：最长的生命周期，单位是秒
- expires:过期的时间。跟max_age类似，但是这个参数要传递一个具体的日期，比如datetime或者是符合日期格式的字符串。如果同时设置了expires和max_age，那么将会使用expires的值作为过期时间
- path：对域名下哪个路径有效，默认是所有路径
- domain：针对哪个域名有效，默认是针对主域名有效
- secure：是否安全，如果设置为True，那么只能在https协议下才可用
- httponly:默认是False，如果是True，那么在客户端不能通过JavaScript进行操作

在settings.py中把是否使用时区改为不使用，把时区改为上海
```
TIME_ZONE = 'Asia/Shanghai'
USE_TZ = False
```
在views.py文件里面
```python
def add_cookie(request):

    response=HttpResponse('设置cookie')

    max_age=60*60*7

    response.set_cookie('username','zhiliao',max_age=max_age)

    return response
```
![[Pasted image 20250222204925.png]]

##### 删除cookie
通过delete_cookie即可删除，实际上删除cookie就是将指定的cookie的值设置为空的字符串，然后将它的过期时间设置为0，也就是关闭浏览器后就过期
```python
def delete_cookie(request):
    response=HttpResponse("删除cookie")
    response.delete_cookie('username')
    return response
```

##### 获取cookie
获取浏览器发送过来的cookie信息，request.COOKIES，再如获取所有的cookie
```python
def get_cookie(request):
    username=request.COOKIES.get('username')
    print(username)
    for key,value in request.COOKIES.items():
        print(key,value)
    return HttpResponse("get cookie")
```
![[Pasted image 20250222210240.png]]

##### django中操作session
通过request.session来操作，常用的方法有
- get:用来从session中获取指定的值
- pop:从session中删除一个值
- keys:从sessions中获取所有的键
- items:从session中获取所有的值
- clear：清除当前这个用户的session数据
- flush:删除session并且删除在浏览器中存储的session_id，一般在注销的时候用得比较多
- set_expiry(value)：设置过期时间，单位秒，要是设置为0，代表只要浏览器关闭，session就会过期，要是不设置这个参数，默认2周后过期，也可以在settings.py中设置SESSION_COOKIE_AGE来配置全局过期时间
- clear_expired:清除过期的session，django不会自动清除过期的session，需要定期手动的清理，也可以在终端执行命令来清除过期的session
```
python manage.py clearsessions
```

```python
def add_session(request):
    request.session['user_id']='zhiliao'
    #获取 username=request.session.get('user_id')
    return HttpResponse("session add")
```
![[Pasted image 20250222211118.png]]

##### 修改session的存储机制
通过设置SESSION_ENGINE来更改session的存储位置，这个可以配置为以下几种方案
1.django.contrib.sessions.backends.db:使用数据库，默认的方案
2.django.contrib.sessions.backends.file：使用文件来存储session
3.django.contrib.sessions.backends.cache：使用缓存来存储session。必须要在settings.py中配置好CACHES并且是需要使用Memcached，而不能使用纯内存作为缓存
4.django.contrib.sessions.backends.cached_db:在存储数据时，先存储到缓存，再存储到数据库。获取数据，是先出缓存中获取
5.django.contrib.sessions.backends.signed_cookies:将session信息加密后存储到浏览器的cookie中。可以设置SESSION_COOKIE_HTTPONLY=True，那么在浏览器中不能通过js来操作session数据，并且还需要对settings.py中的SECRET_KEY进行保密，因为要是被别人知道，就可以进行解密
