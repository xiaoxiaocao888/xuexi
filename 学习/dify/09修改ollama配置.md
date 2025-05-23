![[Pasted image 20250504180638.png]]
这里显示ollama进程绑定到localhost,但是在docker容器中，dify需要通过宿主机IP或docker内部域名访问。
```
sudo vim /etc/systemd/system/ollama.service
```
在`[Service]`部分添加
```
Environment="OLLAMA_HOST=0.0.0.0"  # 新增此行，使 Ollama 监听所有网络接口
```
重启ollama服务
```
sudo systemctl daemon-reload
sudo systemctl restart ollama
```
查看端口进程
```
sudo lsof -i :11434 | grep LISTEN
```
![[Pasted image 20250504181151.png]]