##### selenium
自动化测试工具，可以打开浏览器，然后像人一样去操作浏览器
程序员可以从selenium中直接提取网页上的各种信息

selenium学习网站
```
https://www.byhy.net
```
##### selenium环境搭建
```
pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple selenium
```
下载浏览器驱动，这里是使用谷歌，下载和自己电脑版本一样的

把解压缩的浏览器驱动chromedriver.exe放在了D盘，然后去环境变量添加上D:，这样就不要使用驱动的时候指明驱动的位置
```
https://registry.npmmirror.com/binary.html?path=chrome-for-testing/134.0.6998.88/win64/
```
查看自己电脑的谷歌版本
点击右边三个点--帮助---关于谷歌
![[Pasted image 20250316113801.png]]
##### 验证可以使用成功哦
```python
from selenium import webdriver
#创建webdriver对象，指明使用Chrome浏览器驱动，注意这里是首字母大写
wd=webdriver.Chrome()
#调用webdriver对象的get方法，可以让浏览器打开指定网站
wd.get('https://www.baidu.com')
#程序运行完会自动关闭浏览器，就是很多人说的闪退
#这里加入等待用户输入，防止闪退
input()
```
百度真的打开了哦
![[Pasted image 20250316222133.png]]