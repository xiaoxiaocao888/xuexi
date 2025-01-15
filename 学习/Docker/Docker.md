##### Docker是怎么工作的？
Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上。通过Socket从客户端访问！DockerServer接收到Docker-Client的指令，就会执行这个命令！
![[Pasted image 20241219163656.png]]
##### Docker为什么比VM快？
1.Docker有着比虚拟机更少的抽象层。
2.Docker利用的是宿主机的内核，VM需要是Guest OS。
![[Pasted image 20241219164244.png]]
所以说，新建一个容器的时候，docker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载Guest OS，分钟级别的，而Docker是利用宿主机的操作系统，省略了这个复制的过程，秒级

##### 帮助命令
docker info  显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help  帮助命令
![[微信截图_20241221105140.png]]
![[Pasted image 20241219174523.png]]
##### 镜像命令
docker images 查看所有本地的主机上的镜像
![[Pasted image 20241219175046.png]]
REPOSITORY 镜像的仓库源
TAG  镜像的标签
IMAGE ID  镜像的ID
CREATED  镜像的创建时间
SIZE   镜像的大小
可选项：
-a ,--all 列出所有的镜像
-q,--quiet  只显示镜像的id
![[Pasted image 20241221104251.png]]
##### docker search搜索镜像
![[Pasted image 20241219175742.png]]
可以去官网的文档看命令的相关操作，也可以使用--help命令docker search --help
![[Pasted image 20241219180346.png]]
可选项，通过搜索来过滤
--filter=STARS=3000  搜索出来的镜像就是STARS大于3000的
##### docker pull下载镜像
docker pull mysql 如果不写tag,默认就是下载最新版本
![[Pasted image 20241219193847.png]]
![[Pasted image 20241219194237.png]]
指定版本下载的那个版本一定要存在，可以去仓库那里看有什么版本。
若是下载同一个东西，已经有的部分可以不用下载。（系统自己处理的，看上面的Already exists

##### docker rmi 删除镜像
docker rmi -f 容器id    删除指定的容器
docker rmi -f 容器id 容器id 容器id   删除多个容器，用空格隔开
![[微信截图_20241219205334.png]]

![[Pasted image 20241219212749.png]]
遇到这个问题主要是因为很多镜像源停止服务，需要换的镜像源
1.编辑/etc/docker/daemon.json
```
{"registry-mirrors": ["https://registry.docker-cn.com","http://hub-mirror.c.163.com"]}
```
```
一：{"registry-mirrors": ["https://docker.1panel.live","http://hub.rat.dev"]}
```
2.重启服务
```
[root@localhost ~]# systemctl daemon-reload

[root@localhost ~]# systemctl restart docker
```
3.查看dns解析
```
[root@localhost ~]# dig @114.114.114.114 registry-1.docker.io
```
;; ANSWER SECTION:

registry-1.docker.io. 74 IN A 3.216.34.172

registry-1.docker.io. 74 IN A 34.205.13.154

registry-1.docker.io. 74 IN A 44.205.64.79

4.添加host解析
```
[root@localhost ~]# cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4

::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

3.216.34.172 registry-1.docker.io
```
三、使用docker info 命令检查是否修改成功
显示镜像源的链接就是成功
使用一、2、三成功了

![[Pasted image 20250108211709.png]]
显示超时：
修改配置文件vim /etc/resolv.conf
加入nameserver 114.114.114.114

##### docker流程
![[Pasted image 20250109224924.png]]