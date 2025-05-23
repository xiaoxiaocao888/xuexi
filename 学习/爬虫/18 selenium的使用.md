##### 根据id,class属性，标签查找
```python
from selenium.webdriver.common.by import By
element=wd.find_element(By.ID,'kw') #返回WebElement对象
element.send_keys('通讯\n')
#element.send_keys('通讯')
#element=wd.find_element(By.ID,'go')
#element.click()
wd.find_element(By.CLASS_NAME, 'password').send_keys('sdfsdf') wd.find_element(By.TAG_NAME, 'input').send_keys('sdfsdf') wd.find_element(By.CSS_SELECTOR,'button[type=submit]').click()
```
先引入库，使用find_element就是查找网页源代码里面的标签或属性，By.ID就是查找ID属性，'kw'就是属性值。send_keys就是往那个元素里面输入信息\n就是相当于enter键，这样就开始查询
也可以使用按钮的属性来定位，然后使用.click()点击查询也可以，点击操作点击的是元素正中间的位置
要是查找不到，就会抛出异常

```python
element=wd.find_elements(By.ID,'kw')
```
返回的是一个列表，列表的每个值都是WebElement对象。没有找到返回空列表

###### 要是想判断该元素在不在
```python
from selenium.common.exceptions import NoSuchElementException
try:
   element=wd.find_element(By.ID,'kw')
   element.send_keys('通讯\n')
except NoSuchElementException:
   print('sss')
```

取属性的值
```python
print(element.text)
```

##### 我们也可以通过WebElement选择元素
一样是调用find_element。只是查找范围不同
WebDriver对象选择元素的范围是整个web页面
WebElement对象选择元素的范围是该元素的内部

##### 关闭窗口wd.quit()

##### 使用隐式等待即全局等待
这样是为了防止内容还没有返回，就执行下一步，使得结果出错
在创建wd的时候就加上,参数是指定最大等待时长
那么后续所有的 `find_element` 或者 `find_elements` 之类的方法调用 都会采用上面的策略：
如果找不到元素， 每隔 半秒钟 再去界面上查看一次， 直到找到该元素， 或者 过了10秒 最大时长
```python
wd.implicitly_wait(10)
```
和引入sleep原理类似
```
from time import sleep
sleep(1)
```
对于打开新链接新窗口，我们要使用显示等待
![[Pasted image 20250326211122.png]]
###### 清除掉已经输入的内容
```python
element.clear()
```

###### 通过WebElement对象的get_attribute方法来获取元素的属性值
```python
element=wd.find_element(By.ID,'kw')
print(element.get_attribute('class'))
```
获取元素属性class 的值

###### 获取整个元素对应的HTML
```python
print(element.get_attribute('outerHTML'))#获取整个元素对应的HTML内容
print(element.get_attribute('innerHTML'))#获取元素内部的HTML内容
```

###### 获取输入框里面的文字
```python
print(element.get_attribute('value'))
```

###### 获取元素文本内容2
通过WebElement对象的 `text` 属性，可以获取元素 `展示在界面上的` 文本内容。
但是，有时候，元素的文本内容没有展示在界面上，或者没有完全完全展示在界面上。 这时，用WebElement对象的text属性，获取文本内容，就会有问题。
出现这种情况，可以尝试使用 `element.get_attribute('innerText')` ，或者 `element.get_attribute('textContent')`