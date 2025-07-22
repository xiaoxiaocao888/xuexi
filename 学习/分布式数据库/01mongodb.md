在mongodb shell中可以执行javascript代码，执行会返回一个具有多个文档的数据集
```
var cursor=db.集合.find()
cursor.next()
```

| 方法名(函数)         | 作用                  |
| --------------- | ------------------- |
| hasNext         | 判断是否有更多的文档          |
| next            | 用来获取下一条文档           |
| toArray         | 将查询结果放到数组中          |
| count           | 统计出数据集中文档的数量        |
| limit           | 限定查询数量              |
| skip            | 跳过指定数数量文档           |
| sort            | 对查询结果进行排序           |
| objsLeftlnBatch | 从游标指针当前位置到最后的文档数量   |
| addOption       | 为游标设置辅助选项，修改游标的默认行为 |
| hint            | 为查询强制使用指定索引         |
| explain         | 用于获取查询执行过程报告        |
| snapshot        | 对查询结果使用快照           |
游标的生命周期，包括游标的创建、使用及销毁三个阶段。游标会消耗内存和相关系统资源，游标使用完后应尽快释放资源。
```
db.collection.find(query,projection).addOption()
加选项改变查询的行为。
db.students.find({ age: { $gt: 20 } }).addOption({ maxTimeMS: 5000 }) #限制查询完成的时间。
```
使用游标的步骤：
声明游标；判断游标是否取到数据集结尾；读取数据；关闭游标。
```
var cursor =  db.collection.find(query,projection);
cursor.hasNext() ;
var doc=cursor.Next();
cursor.close();
```

##### 以下情况会让游标被销毁。
① 客户端保存的游标变量不在作用域内。
② 游标遍历完成后，或者客户端主动发送终止消息。
③ 在服务器端10分钟内未对游标进行操作。
④可以通过cursor.noCursorTimeout()来定义游标超时时间
 如：var myCursor = db.users.find().noCursorTimeout()
⑤可以使用 cursor.close()来关闭游标。

在mongo shell中，如果返回的游标结果集未指定var定义的变量，则游标自动迭代20次，即输出前20个文档，超出20的情况则需要输入it来翻页
