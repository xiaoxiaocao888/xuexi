跨域请求：就是浏览器发送的请求协议或IP地址或端口不一样
你看要是没有设置可以跨域，就会跳转不了
![[Pasted image 20250204125426.png]]

设置可以实现跨域请求
法一：自己写个中间件，在响应那里设置请求头范围，`*表示任何地址都可以，也可以自己设置可以访问的IP地址["127.0.0.1",""]
```
@app.middleware("http")

async def MYCORSMiddleware(request:Request,call_next):

    response=await call_next(request)

    response.headers["Access-Control-Allow-Origin"]="*"

    return response
```

法二：使用自定义的中间件
```
from fastapi.middleware.cors import CORSMiddleware

app=FastAPI()

app.add_middleware(

    CORSMiddleware,

    allow_origins="*",

    allow_methods=["GET"],

    allow_headers=["*"],

)
```

在index文件加入以下内容，那么在点击hello world后内容就会发生变化
```
<p>hello world</p>

<script>

    $("p").click(function(){

        $.ajax({

            url:"http://127.0.0.1:8030/user",

            success:function(res){

                console.log(res)

                console.log(res.user)

                $("p").html("hello "+res.user)
            }
        })
    })
```