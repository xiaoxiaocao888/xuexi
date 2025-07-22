1.安装node.js    xhs-mcp-sercer chromedriver
```
pip install node.js
pip install xhs-mcp-server
npx @puppeteer/browsers install chromedriver
```
2.登录小红书账号
C:\Users\iqh\AppData\Local\uv\cache\archive-v0\YZc1sBElAWqnw7sf-Mbtn\Lib\site-packages\xhs_mcp_server\
```
set phone=18069093703 && set json_path=E:\xiaohongshudata&& uvx --from xhs_mcp_server@latest login
uvx --from xhs_mcp_server@latest login --cookies-file E:\xiaohongshudata\xiaohongshu_cookies.json
```
3.可选。在新的命令行输入下面命令，用inspector调试一下，我连接不成功
里面的create_note是发布笔记，create_video_note是发布视频
点击run tool，后台会自动拉起浏览器，打开创作服务平台的发布入口，自动上传图片和填写文字内容并发布出去（内容都是上面调试的内容）
```text
npx @modelcontextprotocol/inspector -e phone=18069093703 -e json_path=E:\xiaohongshudata uvx xhs_mcp_server@latest
```
4.让Qwen3帮我们生成文案，自动发布小红书
登录cherry studio点击模型服务--搜索ModelScope魔塔--在模型下面点击添加--输入Qwen/Qwen3-235B-A22B
上面的密钥，也要去魔塔社区获取，注意在魔塔社区那里要登录上阿里云服务（也要看一下cherry studio页面右上方是不是有红色预警)
点击MCP服务器，添加进我们的小红书服务
```text
"xhs-mcp-server": {
      "name": "xhs-mcp-server",
      "type": "stdio",
      "isActive": true,
      "registryUrl": "",
      "command": "uvx",
      "args": [
        "xhs_mcp_server@latest"
      ],
      "env": {
        "phone": "YOUR_PHONE_NUMBER",
        "json_path": "PATH_TO_STORE_YOUR_COOKIES"
      }
    }
```
添加文生图的能力，来到魔塔社区里面的MCP广场，搜索ModelScope-Image-Generation-MCP,进行安装，输入魔塔社区的APIkey，然后回到cherry studio，MCP服务器--点击同步服务器，输入魔塔社区的APIkey，这样就可以同步魔塔社区安装的插件了。
回到cherry studio的聊天界面，即点击那个聊天图标。把模型切换成Qwen3-235B-A22B,在输入框里面添加上xhs-mcp-server和ModelScope-Image...，就可以进行聊天，让它生成图片，并发布到小红书了。

我都执行了，但是还是显示生成不了图片，也发布不了到小红书，前面可以顺利登录上小红书账号。

还有一个点就是，如果我们本地有使用uv环境创建的项目，要在cherry studio里面使用，我们可以点击MCP服务器，名称描述类型根据自己的项目名写，命令写uv。参数那里写
```
--directory
你的项目完整地址
run
要运行的文件名
```
![[Pasted image 20250607191335.png]]
我们关闭cherry studio，但还是会在任务管理器那里运行着,但是uv进程是关闭的。我们还可以看到下面的python是开启的。可以在状态那里右键勾选命令行，就可以看到命令信息了
![[Pasted image 20250608200536.png]]
所以我们在cherry studio里面也可以使用cmd命令，参数里面填写
```
/c cherrystudio的安装路径\uv --directory 项目完整地址 run 要运行的文件名
```
同理也可以
```
/c 你配置的虚拟环境的完整地址xx\.venv\Scriptes\python.exe 要运行文件的完整地址要有文件名哦
```
![[Pasted image 20250608200637.png]]
别忘记这里啊，点击那个警号，可能就是你的有些命令没有下载，所以使用不了
![[Pasted image 20250609171608.png]]

C:\Users\iqh\AppData\Roaming\uv\python\cpython-3.12.10-windows-x86_64-none\python.exe
C:\Users\iqh\AppData\Local\uv\cache\archive-v0\AcWlhKYWNtxUAWiQbd8_t\Scripts\xhs_mcp_server.exe