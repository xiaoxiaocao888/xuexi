去setting.py文件修改语言系统和时区
```python
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = False
```
把当前用户修改为超级用户，就是把那个is_superuser改为1，is_staff也要改为1

![[Pasted image 20250224223123.png]]

去admin.py文件里面写上一些类，这样才可以在django自带的系统里面进行一些额外内容的操作，这里我们添加一些类，来可以对分类，发布博客，等等
```python
from django.contrib import admin
from.models import BlogCategory,Blog,BlogComment

class BlogCategoryAdmin(admin.ModelAdmin):
    list_display=['name']

class BlogAdmin(admin.ModelAdmin):
    list_display=['title','content','pub_time','category','author']
class BlogCommentAdmin(admin.ModelAdmin):
    list_display=['content','pub_time','author','blog']
admin.site.register(BlogCategory,BlogCategoryAdmin)
admin.site.register(Blog,BlogAdmin)
admin.site.register(BlogComment,BlogCommentAdmin)
```
展示模型的哪个字段，通过` __str__`这个函数  比如提取到某个博客的分类，可以通过名字来展示这个分类
```
def __str__(self):
    return self.name
```

去models.py文件里面增加verbose_name，这样就可以在系统里面显示中文了
增添Meta类就是让那个模型名字也显示中文，写上verbose_name_plural=verbose_name是为了让他的名字有复数的时候还可以显示原来的名字，不然就是显示如博客分类s
```python
class BlogCategory(models.Model):
    name=models.CharField(max_length=200,verbose_name='分类名称')
    def __str__(self):
        return self.name
    class Meta:
        verbose_name='博客分类'
        verbose_name_plural=verbose_name
```
![[Pasted image 20250225090421.png]]