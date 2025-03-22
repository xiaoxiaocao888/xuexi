一般来说我们创建的ORM模型映射到数据库中的表名为 APP名_类名 ,如果想要数据库映射我们自己指定的表明，那么我们可以在Meta类中添加一个db_table的属性。eg
```python
class Book(models.Model):
    name=models.CharField(max_length=100)
    pub_time=models.DateTimeField(auto_now_add=True)
    class Meta:
        db_table='book'
        ordering=['-pub_date']
```
ordering属性设置在提取数据的排序方式，这样查询出来的数据就会根据pub_date字段倒序排序了，这个也是可以设置多个字段排序的。

更多知识可以去下面官网查看的
```
https://docs.djangoproject.com/zh-hans/
```
