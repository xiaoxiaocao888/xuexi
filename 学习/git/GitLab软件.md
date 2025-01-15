GitLab软件，用于搭建自己的代码托管平台
官网地址
```
https://about.gitlab.com/
```
在liunx上面下载gitlab
下载完毕后配置依赖项，以下命令在系统防火墙中打开HTTP HTTPS SSH访问，如果是只从本地网络访问Gitlab，可以跳过这个步骤
```
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```
也可以直接关闭防火墙
```
sudo systemctl stop firewalld
```
初始化Gitlab
```
#配置软件镜像
curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh /bin/bash
#安装,注意那个https是要修改为自己虚拟机的地址，又说是把liunx1改为自己的主机名
sudo EXTERNAL_URL="https://linuxl" yum install -y gitlab-ce
#初始化
sudo gitlab-ctl reconfigure
```
启动gitlab
```
启动
gitlab-ctl start
停止
gitlab-ctl stop
```
访问gitlab，输入网址
```
http://linuxl/users/sign_in
```
要修改liunxl为自己的主机名
打开页面后，刚开始时软件会提供默认管理员账户root，但是密码是随机生成的，在/etc/gitlab/initial_root/password文件里面查找密码
登录成功后，可点击右上角的那个空心圆，点击Edit profile,然后点击下面左边的Password，修改密码

##### 集成idea
在idea里面下载插件GitLab Project
下载成功后对插件进行配置
![[Pasted image 20250114224604.png]]