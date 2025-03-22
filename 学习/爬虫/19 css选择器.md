在css里面我们通过.class值来筛选出class=class值的 元素id 用#id值
```
<head>
  <style>
    .plant{color:red;}
  </style>
</head>
```

###### css选择器同样可以根据tag名，id属性,class属性来选择元素
```python
element=wd.find_element(By.CSS_SELECTOR,'div')
#element=wd.find_elements(By.CSS_SELECTOR,'.animal')选择所有class属性值为animal的元素
```
等价于
```python
element=wd.find_element(By.TAG_NAME,'div')
```

直接子元素就是儿子，后代元素就是子孙
css选择器选择直接子元素
```python
元素1>元素2
```
最终选择的是元素2，且元素2是元素1的直接子元素
也可以是多层，选择的就是最小的直接子元素。中间用>大于号隔开

css选择器选择后代元素
```python
元素1 元素2 元素3
```
最终选择的元素是元素3，要求元素3是元素2的后代，2是1的后代。中间用一个或多个空格隔开

可以混用
```
元素1 元素2>元素3 元素4
```

##### css选择器支持通过任何属性来选择元素，语法是用一个方括号[]
```python
element=wd.find_element('[属性名="属性值"]')
#如果属性名只出现过一次
element=wd.find_element('[属性名]')
```
前面也可以加上标签名的限制，如选择所有标签名为div，且class属性为aa的元素
```python
element=wd.find_element('div[class="aa"]')
```

###### css还可以选择属性值包含某个字符串的元素
如，选择a节点，里面的href属性包含了mimitbeian字符串，可以这样写
```python
a[href*="mimitbeian"]
```
以某个字符串开头
```python
a[href^="http"]
```
以某个字符串结尾
```python
a[href$="gov.cn"]
```

###### css选择器还可以指定选择的元素要同时具有多个属性的限制
```python
div[class=misc][ctype=gun]
```

##### 验证自己写的css选择对不对
在对应的页面，检查，ctrl+f，输入你的选择如div[class="aa"]，要是没有看到高亮，显示为0就是写错了

###### 选择元素1或元素2
```python
元素1,元素2
```
元素里面的关系是不能用括号括起来的

###### 父元素的第n/倒数第n个子节点
```python
p:nth-child(数字)
p:nth-last-child(数字)
p:nth-child(odd/even)#奇数或偶数节点
```
选择的是父元素的第几个/倒数第几个子节点，且子节点标签为p

###### 父元素第n个或倒数第n个某类型的子节点
```python
p:nth-of-type(1)
p:nth-last-of-type(1)
p:nth-of-type(odd/even)#奇数或偶数
```
选择父元素的第1个/倒数第1个p类型的子节点

###### 相邻兄弟节点选择
```
h3+span
```
得到的是与h3相邻兄弟的span

###### 后续所有兄弟节点选择
```
h3~span
```
h3后面所有的兄弟节点span