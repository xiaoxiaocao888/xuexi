##### 要登录才能看到网站内容
在登录的时候打开检查，然后登录之后，点击login查看是get还是Post请求，看携带什么参数
这里用的是POST请求，携带下面的data数据，我们这里使用会话，因为这样可以得到cookie，然后再使用会话进行第二次请求，要是直接使用request的话，会请求不成功，因为这样是从其他请求过来的，用户没有登录。
要是想要用get，那么里面要使用headers={"Cookie":'xxx'}，把cookie数据复制过来，这样可以的原因是cookie本身携带了用户的登录信息
登录->得到cookie->带着cookie去请求到书架url->得到书架上的内容

拿书架数据的时候，就是右击检查，然后找到有数据的文件
```python
import requests
#会话
session=requests.session()
data={
   "loginName":"18614075987",
   "password":"q6035945"
}
url="https://passport.17k.com/ck/user/login"
res=session.post(url,data=data)
#print(res.text)
#print(res.cookies)
#拿书架上的数据
resp=session.get('https://user/17k.com/ck/author/shelf?page=1&appKey=2406394919')
print(resp.json())
#通过cookie
resp=requests.get('https://user/17k.com/ck/author/shelf?page=1&appKey=2406394919',
                 headers={ "Cookie":'xxx'
                  })
```