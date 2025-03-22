自动化程序要使用Xpath来选择web元素，要这样
```python
element=driver.find_element(By.XPATH,"/html/body/div")
```
要选择所有的某一类型的子节点，要用//，如要查找所有标签名为div的元素//div，
要选择 所有的 div 元素里面的 所有的 p 元素，//div//p

`*`是一个通配符，对应任意节点名的元素，如选择所有div节点的所有直接子节点//div/*

#### 根据属性选择
使用这种方式属性值要写全
```python
[@属性名='属性值']
```
eg://p[@class='aa']选择p标签里面class值为aa的元素
`//*[@multiple]` 选择具有multiple属性的所有页面元素
###### 属性值包含字符串
要选择style属性值包含color字符串的页面元素
```python
//*[contains(@style,'color')]
```
要选择style属性值以color字符串开头的页面元素
```python
//*[starts-with(@style,'color')]
```
###### 某类型第几个子元素
`//p[2]` p类型的第2个元素
###### 第几个子元素
`//div/*[2]`父元素为div的第二个子元素
###### 某类型倒数第几个子元素
`//p[last()]`p类型倒数第1个子元素
`//p[last()-1]` p类型倒数第2个子元素
###### 范围选择
如选择option类型第1到第2个子元素
```python
//option[position()<=2] 或//option[opsition()<3]
```
选择class属性为multi的前3个子元素
```python
//*[@class='multi']/*[position()<=3]
```
选择class属性为multi的后三个子元素
```python
//*[@class='multi']/*[position()>=last()-2]
```

###### 组选择，就是或
用竖线隔开，如选择所有的option元素和所有的h4元素
```
//option | //h4
```
等同于css选择器option , h4

###### 选择父节点
某个元素的父节点用/..表示，应用与某个元素没有特征可直接选择，但是子节点有特征
如选择id为china的节点的父节点
`//*[@id='china']/..`
还可以继续往上找
`//*[@id='china']/../../..`

###### 兄弟节点选择
选择后续兄弟节点`following-sibling::`
如要选择class为single的元素的所有后续兄弟节点
`//*[@class='single']/following-sibling::*`
选择后续节点中的div节点
`//*[@class='single']/following-sibling::div`
选择最靠近的兄弟节点
`//*[@class='single']/preceding-sibling::*[1]`
第二靠近就写2

###### 要在某个元素内部使用xpath选择元素，要在xpath表达式前面加个.
`elements=china.find_elements(By.XPATH,'.//p')

