表单的Field和模型中的Field基本上是一模一样的，而且表单中需要验证的数据，也是模型中需要保存的，那么这个时候我们可以将模型中的字段和表单中的字段进行绑定
比如有一个article模型,我们要写一个article表单，可以这样来写
```
from django import forms
class MyForm(forms.ModelForm):
      class Meta:
            model=Article
            fields="__all__" #继承所有的字段
            fields=['title','content']#继承特定的字段
            exclude=['title']#除了title这个字段不继承，其他都继承
```
指定了auto_now_add=True,那么在表单中可以不用传入这个字段
blank=True，只是表单验证时允许为空，不代表数据库可以为空

##### 自定义错误消息
在Meta类中，定义error_messages，把相应的错误消息写道里面
```
error_messages={
   'title':{
        'max_length':'最多不能超过10个字符'
    }
}
```

##### save方法
在验证完成后直接调用save方法，就可以将这个数据保存到数据库中
```
if form.isvalid():
    form.save()
```
