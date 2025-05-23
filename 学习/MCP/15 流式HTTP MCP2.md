回到主目录，修改配置文件pyproject.toml
这是一个典型的Python项目配置文件，用于定义项目的构建、依赖和分发规则。下面依次解释
- 声明项目构建所需的工具链
     - requires：指定构建依赖为setuptools，版本>=61.0，和wheel，用于生成分发包
     - build-backend:使用setuptools作为构建后端，负责编译源码和生成安装文件
- 项目元数据(定义项目核心信息与依赖关系)
     - name项目名称，用于PyPI发布和安装时的标识
- 命令行入口。创建可执行命令 可以直接在终端执行python -m mcp_weather_http
- 目录结构配置。自定义源码目录结构。将根目录映射到src文件夹，即源码存放于src/下
- 指示setuptools在src/目录下查找Python。
`setuptools` 是 Python 生态中​**​最核心的包构建和分发工具​**​，用于将 Python 代码打包成可安装的格式（如 `.whl` 或 `.egg`），并管理项目依赖关系。
```Plaintext
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mcp-weather-http"
version = "1.1.0"
description = "输入OpenWeather-API-KEY，获取天气信息。"
readme = "README.md"
requires-python = ">=3.9"
dependencies = [
    "httpx>=0.28.1",
    "mcp>=1.8.0",
]

[project.scripts]
mcp-weather-http = "mcp_weather_http:main"

[tool.setuptools]
package-dir = {"" = "src"}

[tool.setuptools.packages.find]
where = ["src"]
```

##### HTTP流式传输MCP服务器开启与测试
先开启流式HTTP MCP服务器，然后再开启Inspector
在stdio模式下是同时开启
```
在mcp-weather-http目录下
uv run ./src/mcp_weather_http/server.py --api-key OPENWEATHERKEY
npx -y @modelcontextprotocol/inspector uv run main.py --api-key YOUR_KEY 
# 我的代码里面已经设置了密钥了，所以这里可以不用加--api-key参数
```
然后打开Inspector，在浏览器输入http://127.0.0.1:6274
选择HTTP流式模式，选择默认地址http://localhost:3000/mcp，点击connect,点击List Tools，点击get-weather，输入地名，若是能返回天气，说明服务器运行正常。
![[Pasted image 20250521222541.png]]
![[Pasted image 20250522084921.png]]
##### 流式HTTP MCP服务器异地调用(测试)
接下来即可在异地环境（也可以是本地）通过HTTP方式调用MCP服务了
在本地安装Cherry Studio。cherry studio是一个功能非常完整的MCP这样的客户端，在其中可以接入各式各样不同的MCP工具
进行使用：需要保持远程流式HTTP MCP服务器开启。
点击左下方的设置--点击MCP服务器--点击添加服务器--输入服务器名称mcp-weather-http，选择流式传输类型，地址为http://localhost:3000/mcp(若是远程的，里面的localhost可以改为那个服务器地址)--点击保存(不成功，可以多试一次)出现API验证错误，就点击设置看看里面的API是否配置了，按提示配置即可
若页面显示服务器更新成功，则表示已经连接上MCP服务器。回到对话页面，选择MCP服务器(就是刚刚的mcp-weather-http)，然后就可以进行天气查询。
![[Pasted image 20250522162442.png]]
##### 流式HTTP MCP服务器发布流程
测试完成，即可上线发布，发布到pypi平台
打包上传：
```
回到项目主目录mcp-client
uv pip install build twine #打包和上传的库
python -m build  #使用setuptools构建分发包
python -m twine upload dist/*   #上传到PyPI
```
去那个链接进行注册，生成API key，才可以上传到自己的平台
先去注册登录，下面是二重验证的步骤
下载uv pip install pyotp
输入运行以下代码，然后把输出6位数字写入到网站，完成验证
```python
import pyotp
key='粘贴网站上面那32位的英文数字'
totp=pyotp.TOTP(key)
print(totp.now())
```
然后去C:\Users\Administrator目录下新建.pypirc文件,password为令牌值
```
[pypi]
username = __token__
password =xxx
```
![[Pasted image 20250522181235.png]]
显示这个错误可能是项目名和其他人上传者重复了，我们去项目mcp-client目录下的pyproject.toml修改里面的name值为mcp-weather-http-lqh
执行rd /s /q dist删除前面生成的内容，重新执行上面的命令就可以啦
![[Pasted image 20250522181506.png]]
可以点击该链接看到刚刚发布的库
![[Pasted image 20250522181549.png]]
本地安装(可以下载其他人发布的库)
```
pip install mcp-weather-http
```
开启服务：输入openweatherapi
```
uv run mcp-weather-http --api-key YOUR_API_KEY
```
然后即可打开浏览器输入`http://localhost:3000/mcp`测试连接情况
也可以使用cherry studio进行连接，和上面的流程一样