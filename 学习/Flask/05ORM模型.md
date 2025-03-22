对象关系映射，简称ORM，是一种可以用python面向对象的方式来操作关系型数据库的技术，具有可以映射到数据库表能力的python类我们称之为ORM模型。一个ORM模型与数据库中一个表相对应，ORM模型中的每个类属性分别对应表的每个字段，ORM模型的每个实例对象对应表中每条记录。
Flask-SQLAlchemy底层支持SQLite、MySQL、Oracle、PostgreSQL等关系型数据库，但针对不同的数据库，ORM模型代码几乎一模一样，只需修改少量代码,如对下方代码的pymysql进行替换即可
```
DB_URI = f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4"
```

创建类对象与提交
所有ORM模型必须是db.Model的直接或者间接子类。通过__tablename__属性，指定User模型映射到数据库中表的名称。只有使用db.Column(这里的db是这个db=SQLAlchemy(app))定义的类属性，才会被映射到数据库表中成为字段。
```
class User(db.Model):
    __tablename__="user"
    id=db.Column(db.Integer,primary_key=True,autoincrement=True)
    #varchar类型,null=0或为假
    username=db.Column(db.String(100),nullable=False)
    password=db.Column(db.String(100),nullable=False)

with app.app_context():
    db.create_all()
    # 创建 User 实例
    user = User(username="lili", password='111')
    # 将实例添加到会话中
    db.session.add(user)
    # 提交会话以保存更改到数据库
    try:
        db.session.commit()
        print("数据插入成功")
    except Exception as e:
        print(f"数据插入失败: {e}")
        # 发生错误时回滚会话
        db.session.rollback()
    finally:
        # 关闭会话
        db.session.close()
```

在 Flask 里，`with app.app_context()` 是一个非常重要的上下文管理器。上下文是一种保存应用运行时信息的机制，主要分为应用上下文（`Application Context`）和请求上下文（`Request Context`）。应用上下文保存了与应用实例相关的信息，请求上下文则保存了与当前请求相关的信息。
Flask 是一个多线程或者多进程的 Web 框架，可能同时处理多个请求。为了确保每个请求能够正确地访问应用实例和相关资源，Flask 使用上下文来管理这些信息。当你在一个脚本或者函数中需要使用 Flask 应用的某些功能，但又不在请求处理函数内部时，就需要手动创建应用上下文。

##### 向表中添加数据
```
@app.route("/user/add")
def add_user():
    #1.创建ORM对象
    user = User(username="xiaoxiao", password='444')

    #2.将ORM对象添加到db.session中
    db.session.add(user)

    #3.将db.sesssion中的改变同步到数据库中
    db.session.commit()

    return "用户创建成功"
```

##### 查询表中数据
```
@app.route("/user/query")

def query_user():

    #1.get查找：根据主键查找
    # user = db.session.get(User, 1)
    # print(f"{user.id}--{user.username}--{user.password}")

    #2.filter_by查找
    users=User.query.filter_by(username="xiaoxiao")
    user2=User.query.all()
    user3=User.query.first()        user4=User.query.filter(and_(User.username=="lili",User.id>3)).all()
    user5=User.query.order_by(User.id.desc()).all()
    user6=User.query.count()
    print(user2)
    print(user3)
    for user in users:
        print(user.username)
    for user in user4:
        print(user.password)
    print(user5)
    print(f"用户的总数量是{user6}")
    return "数据查询成功"
```
![[Pasted image 20250208204232.png]]

##### 修改表中数据
```
@app.route("/user/update")
def update_user():
    user=User.query.filter_by(username="lili").first()
    user.password="2222333"
    db.session.commit()
    print(user.password)
    return "数据修改成功"
```

##### 删除表中数据
```
@app.route("/user/delete")
def delete_user():
    #1.查找
    user7=User.query.get(1)

    #2.从db.session中删除
    db.session.delete(user7)

    #3.将db.session中的修改，同步到数据库中
    db.session.commit()
    return "删除成功"
```

建立一对多的关系，就是外键关联
法一，在其中一个类写，那么会在User类里面就会自动添加一个aerticles字段
```
author=db.relationship("User",backref="articles")
```
法二：
```
author=db.relationship("User",back_populates="articles")
#在User类也要写
articles=db.relationship("Article",back_populates="author")
```
这样在查找时就方便
```
@app.route("/article/query")
def query_article():
    user=User.query.get(2)
    for article in user.articles:
        print(article.title)
    return "文章查找成功"
```
article=Article(title="Flaskxue",content="ddss")
article.author就会相当于User.query.get(article.author_id)

模型映射成表的方式，先下载好对应的插件
```
pip install flask-migrate
```
在代码中加入
```
from flask_migrate import Migrate
db = SQLAlchemy(app)
migrate=Migrate(app,db)
```
ORM模型映射成表的三步

1.flask db init :这步只需要执行一次

2.flask db migrate :识别ORM模型的转变，生成迁移脚本

3.flask db upgrade :运行迁移脚本，同步到数据库中

如果有多个py文件，可以先执行这个，文件名改为对应的
```
 $env:FLASK_APP = "shujuku.py"
```