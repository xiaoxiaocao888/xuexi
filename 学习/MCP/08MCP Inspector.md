这是一个debug工具，使用它可以快捷地调用各类server，并测试其功能
安装nodejs
```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs
```
验证是否安装成功
```
npx -v
```
运行Inspector
```
npx -y @modelcontextprotocol/inspector uv run server.py
```
然后即可在本地浏览器查看当前工具运行情况：http://127.0.0.0.1:5173/#resources
![[Pasted image 20250508163037.png]]