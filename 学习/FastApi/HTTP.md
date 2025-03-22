1.什么是请求头请求体，响应头响应体
2.URL地址包括什么
3.get请求和post请求到底是什么
4.Content-Type是什么


HTTP协议是超文本传输协议，是用于万维网WWW服务器与本地浏览器之间传输超文本的传送协议。
![[Pasted image 20250118103600.png]]

##### HTTP特性
1.基于TCP/IP协议
2.基于请求-响应模式（先有请求，再有响应）
3.无状态保存。就是对发送过的请求或响应都不做持久化处理，即发出请求后，服务器不会知道客户端之前是否访问过，访问过多少次等。
4.短连接。
HTTP1.0默认使用短连接，浏览器和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。HTTP1.1起，使用长连接，客户端和服务器的HTTP首部的Connection都要设置为keep-alive，长连接指的是复用TCP连接，多个HTTP请求可以复用同一个TCP连接

##### HTTP请求与响应格式
![[Pasted image 20250118111639.png]]
socket封装一整套网络协议
get请求和post请求的区别是发送的格式不一样，get提交的数据放在URL后面，以？分割URL和传输数据，参数之间以&相连，post请求把请求内容放在请求体
![[Pasted image 20250118113607.png]]
请求首行里面有请求方式，请求路径+参数，协议
请求头与请求体之间要用空行来隔开
服务器根据请求头的content-type来解析数据，因为数据可能是json格式也可能是其他格式。
响应头的content-type表明响应体的格式是什么
200是状态码，OK表示正常的访问
一个完整的URL包括：协议、ip、端口、路径、参数
例如https://www.baidu.com/s?wd=yuan 其中https是协议，www.baidu.com是域名可转换为ip，端口默认80，/s是路径，参数是wd=yuan

send发送的响应信息，一定要有HTTP/1.1 200 ok响应头，两个\r\n就是换行，区分响应头响应体
![[Pasted image 20250118121847.png]]
![[Pasted image 20250118121727.png]]

在Postman软件里面，选择get，复制浏览器上127那个链接到软件，点击Send，也可以看到内容
![[Pasted image 20250120110718.png]]
若是在Params里面输入数据，数据会同步到链接上面，也可以看到请求那里是有a=1,b=2
![[Pasted image 20250120111854.png]]
![[Pasted image 20250120112006.png]]

数据直接显示在后面是不安全的，所以我们可以使用post，这样数据就是放在请求体里面
选择表单的格式就是=
![[Pasted image 20250120112349.png]]
![[Pasted image 20250120112406.png]]
使用json格式就是字典形式
![[Pasted image 20250120112617.png]]
![[Pasted image 20250120112651.png]]

```
conn.send(b"HTTP/1.1 200 ok\r\nserver:yuan\r\ncontent-type:text/plain\r\n\r\n<h1>hello word</h1>")
```
这句话就是响应体的格式是纯文本格式，所以返回来的就是这样
![[Pasted image 20250120114322.png]]
若是content-type:text/html，就会进行渲染
![[Pasted image 20250120114434.png]]

content-type:application/json这个就是响应体的格式是json格式，那么发过去的内容也要是json格式
在Postman里面打开它的控制台:点击左上角的三个杠--view--show，就可以了，点击send发送就可以看到内容，页面中间显示数据那里raw就是显示原生数据，Pretty就是对数据进行结构化，更加美观
![[Pasted image 20250120115344.png]]