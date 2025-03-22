带参数的url,对参数加一个<>
```
@app.route("/blog/<blog_id>")
def blog_detail(blog_id):
    return "您访问的博客是：%s" % blog_id
```
![[Pasted image 20250207193315.png]]

##### 查询字符串的方式传参
/book/list 默认返回第一页的数据  /book/list?page=2 获取第二页的数据
```
@app.route('/book/list')

def book_list():

    #args是arguments的缩写，是参数的意思

    page=request.args.get("page",default=1,type=int)

    return f"您获取的是第{page}的图书列表！"
```