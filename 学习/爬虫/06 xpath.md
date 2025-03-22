**xpath** 是在XML文档中搜索内容的一门语音
html是xml的一个子集
我们需要安装lxml模块来使用xpath
```
pip install lxml
```
xml的属性之间是父子或同胞关系
/表示层级关系，第一个/是根节点
.text()是拿标签里面的内容
```
from lxml import etree
xml="""xlm文件的内容"""
tree=etree.XML(xml)
result=tree.xpath("/book/name/text()")#拿取book的子标签name的内容
result=tree.xpath("/book/author//nick/text()")#//表示后代，拿取book的author的后代的所有nick标签的内容
result=tree.xpath("/book/*/nick/text()")#*表示任意节点，只一层
```

xpath的顺序是从1开始数的,[]表示索引
`result=tree.xpath("/book/name[1]/text()")`获取第一个name的值
[@xxx='xxx']对属性进行筛选`result=tree.xpath("/book/name[@href='1']/text()")`获取name标签里面href为1的name值

##### 相对查找.
result1=result.xpath("./a/text()")` 

##### 拿到属性值：@属性名
result2=reslut.xpath("./a/@href")

##### 复制网页某个地方的xpath
右击检查，选中Elements，点击最左边的箭头，然后点击页面的你想要的部分，在对应的html那里右击，点copy--copy xpath
![[Pasted image 20250313165846.png]]