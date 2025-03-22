在函数中声明request类型的参数，Fast API就会自动传递Request对象给这个参数，就可以获取到对应信息
代码
```
@app06.post("/items")

async def items(request:Request):

   print("URL",request.url)

   print("客户端IP地址",request.client.host)

   print("客户端宿主",request.headers.get("user-agent"))

   print("cookies",request.cookies)

   return{

      "URL":request.url,

      "客户端IP地址":request.client.host,

      "客户端宿主":request.headers.get("user-agent"),

      "cookies":request.cookies
   }
```
![[Pasted image 20250121222148.png]]

测试cookie，要在postman软件上测试
![[Pasted image 20250121222229.png]]