拉取项目到本地
```
git clone https://github.com/searxng/searxng-docker.git
```
修改配置文件docker-compose
把里面port那里的127.0.0.1删除掉
修改searxng文件夹里面的settings文件，主要是修改内容里面的密码，什么都可以，然后limit改为false，添加一些搜索引擎，这样就可以免费，具体全部内容如下(放在最后面)
我们刚开始是克隆到本地window的，使用xftp上传到服务器
```
sudo systemctl start docker
cd ~/searxng-docker
sudo docker compose up -d
```
执行完之后，就会自动拉取镜像。我们可以去前端访问
http://127.0.0.1：8080/search  可以进行搜索
我是执行了sudo docker compose ps 发现了我刚开始输入的端口号不对，才访问不了前端页面
![[Pasted image 20250529211032.png]]
![[Pasted image 20250529210913.png]]
https://r.jina.ai/要读取的网页地址 把网页的内容转换为markdown格式，好让机器读懂
![[Pasted image 20250529211913.png]]
登录这个网站，可以使用里面一些免费的模型
https://openrouter.ai/models
在虚拟机上面执行sudo ufw allow 8080/tcp 开放端口
我运行就是会卡着，没有数据出来
```
# see https://docs.searxng.org/admin/settings/settings.html#settings-use-default-settings
use_default_settings: true

engines:
  # 启用默认禁用的引擎
  - name: bing
    disabled: false

  - name: 360search
    engine: 360search
    shortcut: 360so
    disabled: false
  
  - name: baidu
    engine: baidu
    shortcut: baidu
    disabled: false
  
  - name: iqiyi
    engine: iqiyi
    shortcut: iq
    disabled: false

  - name: acfun
    engine: acfun
    shortcut: acf
    disabled: false


  # 禁用默认启用的引擎
  - name: arch linux wiki
    engine: archlinux
    disabled: true
  - name: duckduckgo
    engine: duckduckgo
    disabled: true
  - name: github
    engine: github
    shortcut: gh
    disabled: true
  - name: wikipedia
    engine: wikipedia
    disabled: true
    

server:
  # base_url is defined in the SEARXNG_BASE_URL environment variable, see .env and docker-compose.yml
  secret_key: "888"  # change this!
  limiter: false  # can be disabled for a private instance
  image_proxy: true
ui:
  static_use_hash: true
redis:
  url: redis://redis:6379/0

search:
  safe_search: 0
  autocomplete: ""
  default_lang: ""
  formats:
    - html
    - json
    - csv
    - rss
ratelimit:
    enabled: true
    # 调整每秒允许的请求数
    per_second: 5  
    # 调整每分钟允许的请求数
    per_minute: 60
```