ORM模型一般放在app的models.py文件中，每个app都可以拥有自己的模型。要把模型映射到数据库中，要把app放在settings.py的INSTALLED_APP中进行安装。以下是一个简单的书籍ORM模型
```python
from django.db import models

class Book(models.Model):

    name=models.CharField(max_length=100)

    author=models.CharField(max_length=20)

    pub_time=models.DateTimeField(auto_now_add=True)

    price=models.FloatField(default=0)
```
这个模型继承自django.db.models.Model，这是每个ORM模型都要继承的
这个模型没有定义主键，但是这样会自动生成一个自动增长的int类型的主键id

##### 映射模型到数据库中
1.在settings.py中，配置好DATABASES，做好数据库相关的配置
2.在app中的models.py中定义好模型，这个模型必须继承自django.db.models.
3.将这个app添加到settings.py的INSTALLED_APP中
4.在命令行终端，进入到项目所在的路径，然后执行命令生成迁移脚本文件
```
python manage.py makemigrations
```
5.执行命令将迁移脚本文件映射到数据库中
```
python manage.py migrate
```

![[Pasted image 20250220211110.png]]

##### 增
创建完对象后，调用模型的save方法，django会自动的将这个模型转换成sql语句，然后存储到数据库中
在views.py文件中
```python
def add_book(request):

    book=Book(name='三国演义',author='罗贯中',price=100)

    book.save()

    return HttpResponse("图书插入成功！")
```
在urls.py文件中
```
path('book/add',book_views.add_book,name='add_book')
```
这样就新增成功了
![[Pasted image 20250220215457.png]]

##### 查询
查找数据都是通过模型下的objects对象来实现的
1.查找所有数据
```python
def query_book(request):

    books=Book.objects.all()

    for book in books:

        print(book.id,book.name,book.pub_time,book.price)

    return HttpResponse("查询成功！")
```

2.数据过滤
```
books=Book.objects.filter(name='三国演义')
```

3.获得单个对象
返回第一个满足条件的对象，用get方法，但是要是没有找到满足条件的对象，会抛出异常，而filter没有找到是返回一个空列表
```python
# book=Book.objects.get(name='三国演义')
try:

        book=Book.objects.get(name='三国演义1')

        print(book.name)

    except Book.DoesNotExist:

        print("图书不存在")
```

4.数据排序
实现倒序排序，就是在字段前面加一个-号
```
books=Book.objects.order_by("pub_date")
books=Book.objects.order_by("-pub_date")
```

##### 改
先查询再修改，再save
```python
def update_book(request):

    book=Book.objects.first()

    book.name='西游记'

    book.save()

    return HttpResponse('修改成功')
```

##### 删
先查询，再delete
```python
def delete_book(request):

    book=Book.objects.filter(name='西游记')

    book.delete()

    return HttpResponse("删除成功")
```