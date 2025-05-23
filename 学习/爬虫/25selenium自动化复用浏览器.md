1.关闭所有chrome浏览器
2.获取启动路径，属性那里
```
C:\Program Files\Google\Chrome\Application
```
3.配置环境变量，在path里面新建上面到路径
4.启动命令行,输入下面内容，就会打开浏览器
```
chrome --remote-debugging-port=9222
```
5.验证http://localhost:9222/


每次都要把浏览器关闭，然后再cmd上执行
```
chrome.exe --remote-debugging-port=9222 --user-data-dir="D:\selenium\ChromeProfile"
```
然后再运行程序
```python
options = webdriver.ChromeOptions()  
options.debugger_address = "127.0.0.1:9222"  # 返回远端devtools实例的地址  
# 使用ChromeDriverManager自动下载ChromeDriver，并获取其路径  
driver_path = ChromeDriverManager().install()  
print(driver_path)  
# 创建Service对象，并指定ChromeDriver的路径
service = Service(executable_path=driver_path)  
# 使用Service对象创建WebDriver  
driver = webdriver.Chrome(service=service,options=options)  
url="https://www.bilibili.com/"
driver.get(url=url)
```