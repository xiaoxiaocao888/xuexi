##### 查询操作
在python.py文件里面加入代码
注意这里的tortoise是要用异步的，所以在函数前面要加入async，在数据操作前面那里要加上await，这样就是协程异步
（1）查新所有
```
async def getAllStudent():
    students=await Student.all() 
#返回数据类型是Queryset[Student(),Student(),Student()]

    print("students",students)
    for stu in students:
        print(stu.name,stu.sno)
    print(students[0].name)
    return{
        "操作":"查看所有的学生"
    }
```
![[Pasted image 20250201111959.png]]

（2）过滤查询filter 返回数据类型是`Queryset[Student(),],Queryset[Student(),Student()]
```
    students1=await Student.filter(name="rain")
    students2=await Student.filter(clas_id=14)
    print("students1",students1)
    print("students2",students2)
```
![[Pasted image 20250201113717.png]]

(3)过滤查询 get方法：返回模型类型对象
```
students3=await Student.get(id=6) #Student()
    print(students3.name)
```
![[Pasted image 20250201114512.png]]

（4）模糊查询
```
students4=await Student.filter(sno__gt=2001)
students5=await Student.filter(sno__in=[2001,2002])
```
`__gt是表示大于，也有__range表示范围，__lt表示小于

（5）values查询
```
students6=await Student.all().values("name","sno")
print(students6)#[{'name':'rain,'sno':'2001'},{},{}]
```

(6)一对多查询，多对多查询
```
avlin=await Student.get(name="avlin")
    print(avlin.name)
    print(await avlin.clas.values("name"))
    students8=await Student.all().values("name","clas__name","courses__name")
    print(students8)
    print(await avlin.courses.all().values("name","teacher__name"))
```
teacher__name就是courses的外键表的name
![[Pasted image 20250202175252.png]]
如果前面有对路径后面的参数做类型限制为int，那么访问/index可能就会失败

##### 使用bootstrap组件
在index.html文件的`<head></head>里面加上<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">

然后就可以使用它的组件信息了
![[Pasted image 20250202143551.png]]
![[Pasted image 20250202143758.png]]

##### 添加操作
写入一个校验模型
```
class StudentIn(BaseModel):
    name:str
    pwd:str
    sno:int
    clas_id:int
    courses:List[int]=[]
    @validator("name")
    def name_must_alpha(cls,value):
        assert value.isalpha(),'name must be alpha'
        return value
```

```
@student_api.post("/")
async def addStudent(student_in:StudentIn):
    #插入到数据库
    #方式一#student=Student(name=student_in.name,pwd=student_in.pwd,sno=student_in.sno,clas_id=student_in.clas_id,)
#await student.save() #插入到数据库student表
    #方式二
student=await Student.create(name=student_in.name,pwd=student_in.pwd,sno=student_in.sno,clas_id=student_in.clas_id,)
    # return{
    #     "操作":"添加一个学生"
    # }
    return student
```
这两种方式都可以看到数据库里面添加到了数据

 多对多关系的绑定
```
    choose_courses=await Course.filter(id__in=student_in.courses)
    await student.courses.clear()#清除原先的绑定
    await student.courses.add(*choose_courses)#加*是因为数据是列表
```
这样就可以看到course也插入到了数据库里面

##### 更新操作
```
@student_api.put("/{student_id}")
async def updateStudent(student_id:int,student_in:StudentIn):
    data=student_in.dict()
    print("data",data)
    courses=data.pop("courses")
    await Student.filter(id=student_id).update(**data)
    #await Student.filter(id=student_id).update(name=student_in.name,,,)

#设置多对多的选修课,这是要单独做的
    edit_stu=await Student.get(id=student_id)
    choose_courses=await Course.filter(id__in=courses)
    await edit_stu.courses.clear()
    await edit_stu.courses.add(*choose_courses)
    return edit_stu
```

##### 删除操作
```
@student_api.delete("/{student_id}")
async def updateOneStudent(student_id:int):
    deletCount=await Student.filter(id=student_id).delete()
    if not deletCount:
        raise HTTPException(status_code=404,detail=f'主键为{student_id}的学生不存在')
    return{}
```