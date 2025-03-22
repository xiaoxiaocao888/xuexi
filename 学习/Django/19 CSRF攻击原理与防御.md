##### CSRF攻击原理
CSRF(Cross Request Forgery,跨域请求伪造)是一种网络的攻击方式
如果你访问了一个别有用心或病毒网站，这个网站可以在网页源代码中插入js代码，使用js代码给其他服务器发送请求，因为在发送请求的时候，浏览器会自动的把cookie发送给对应的服务器，这时候相应的服务器不知道这个请求是伪造的，就被欺骗了，从而达到在用户不知情的情况下，给某个服务器发送了一个请求

##### 防御CSRF攻击
在用户每次访问有表单的页面的时候，在网页源代码中加一个随机的字符串叫做csrf_token，在cookie中也加入csrf_token字符串，以后给服务器发送请求的时候，必须在form中以及cookie中都携带csrf_token,服务器只有检测到cookie中的csrf_token和form中的csrf_token都匹配（不是相等），才认为这个请求是正常的，否则就是伪造的。
在django中，防御CSRF攻击，要做两步工作：一在settings.py的MIDDLEWARE中添加CsrfViewMiddleware中间件（默认是有的）；二是在模板代码中添加一个input标签，加载csrf_token.
模板代码
```html
<input type="hidden" name="CsrfMiddlewaretoken" value="{{csrf_token}}"/>
```
或者是直接使用csrf_token标签，来自动生成一个带有csrf_token的input标签
```
{% csrf_token %}
```
要是页面中显示403错误，写有CSRF，说明你忘记些模板代码了
