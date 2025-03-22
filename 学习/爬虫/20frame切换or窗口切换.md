##### 切换到frame
![[Pasted image 20250319203441.png]]
在html语法中,frame元素或者iframe元素的内部会包含一个被嵌入的另外一份html文档
我们使用selenium打开一个网页，操作的范围是不包括被嵌入的html文档内部的内容

要操作被嵌入的HTML文档的元素，就必须切换操作范围 到被嵌入的HTML文件
```python
wd.switch_to.frame(frame_reference)
#wd.switch_to.frame('innerFrame')
```
frame_reference参数可以直接写为id或name的值，也可以填写frame所对应的WebElement对象
```python
wd.switch_to.frame(wd.find_element(By.TAG_NAME,"iframe"))
```

切换到被嵌入的文档后又想操作主HTML文档，就得转换到原来的主HTML
```python
wd.switch_to.default_content()
```

当前窗口的标题栏文本
```python
print(wd.title)
```

##### 切换到新窗口
使用selenium写自动化程序时点击了链接或按钮，打开了新的窗口，如果想在新窗口上进行操作，我们需要切换到新窗口
```python
wd.switch_to.window(handle)
```
WebDriver对象有window_handles属性，是一个列表，包含了当前浏览器里面所有的窗口句柄(类似id)
```python
for handle in wd.window_handles:
    # 先切换到该窗口
    wd.switch_to.window(handle)
    # 得到该窗口的标题栏字符串，判断是不是我们要操作的那个窗口
    if 'Bing' in wd.title:
        # 如果是，那么这时候WebDriver对象就是对应的该该窗口，正好，跳出循环，
        break
```
如果要回到原来的窗口，也是可以用for循环
当然我们也可以在开始的时候保留好句柄
```python
# mainWindow变量保存当前窗口的句柄
mainWindow = wd.current_window_handle

#通过前面保存的老窗口的句柄，自己切换到老窗口
wd.switch_to.window(mainWindow)
```