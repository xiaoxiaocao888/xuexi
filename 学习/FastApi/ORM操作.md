ORM（Object-Relational Mapping，对象关系映射）是一种编程技术，用于将对象模型（面向对象编程中的类和对象）与关系模型（数据库中的表和记录）之间进行映射。它的主要目的是让开发者能够使用面向对象的方式来操作关系数据库，而无需直接编写 SQL 语句。

fastapi是一个很优秀的框架，但是缺少一个合适的orm，官方代码使用的是sqlalchemy,Tortoise ORM是受Django启发的易于使用的异步ORM（对象关系映射器），模拟sql操作
Tortise ORM 目前支持以下数据库：
PostgreSQL>=9.4（使用asyncpg)
SQLite(使用aiosqlite)
MySQL/MariaDB(使用aiomysql或使用asyncmy)

下载
```
pip install tortoise-orm
```

fields把所有字段都封装在这个文件里面

create database fastapi;

##### vscode与mysql连接
先在扩展那里下载mysql，下载成功后在主页会有MySQL点击‘+’，新建连接，写入IP地址127.0.0.1，之后写入用户名密码，即可连接成功，可看到对应的数据库
![[Pasted image 20250131205041.png]]

安装Database Client插件，点击database，新建连接，写入对应信息，可以直接对表进行操作
![[Pasted image 20250201105211.png]]

aerich是一种ORM迁移工具，需要结合tortoise异步orm框架使用，安装aerich，若是有缺失什么模块，就下载缺失的模块就可以啦
```
pip install aerich
```

对aerich进行初始化配置，只需要使用一次，完成后生成pyproject.toml(保存配置文件路径)和migrations文件夹(存放迁移文件)
```
aerich init -t settings.TORTOISE_ORM
```


初始化数据库，一般情况下只用一次
```
aerich init-db
```

然后在连接名称那里右击鼠标点击Refresh就刷新，就可以看到对应的表了
![[Pasted image 20250131215639.png]]

##### 新增字段
在models.py文件里面加上addr字段后保存，输入
```
aerich migrate
```
也可以输入
```
aerich migrate --name 修改名称
```
就是会生成新的文件，在migrations\models文件下面的xxx_update.py文件，然后再在终端输入
```
aerich upgrade
```
才会把addr真正写入到数据库
![[Pasted image 20250131220725.png]]

若是想返回到先前的版本，就输入
```
aerich downgrade
```
刷新一下就看不到先前新增的addr字段了
![[Pasted image 20250131221218.png]]

查看历史迁移数据
```
aerich history
```
![[Pasted image 20250131221737.png]]