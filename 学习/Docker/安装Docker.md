###### 环境查看
系统内核是3.10以上的
![[Pasted image 20241218114219.png]]
系统版本
![[Pasted image 20241218114400.png]]
安装：帮助文档
![[Pasted image 20241218152816.png]]
###### 1.卸载旧版本
```console
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
###### 2.需要的安装包
```
yum install -y yum-utils
```
![[Pasted image 20241218154800.png]]
###### 3.设置镜像仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo --配置阿里云仓库
执行不成功，发现是ascii错误，检查发现是/usr/bin/yum-config-manager的编码是ascii，`file -bi /usr/bin/yum - config - manager` 检查文件编码,要把文件编码修改为utf-8,
1. 首先用`vim`打开`/usr/bin/yum - config - manager`文件，即`vim /usr/bin/yum - config - manager`。
2. 进入`vim`后，在`vim`的命令模式（按`Esc`键确保进入命令模式）下输入`:set bomb`，这一步是设置文件格式为二进制，以便后续进行编码转换操作。
3. 接着在`vim`的命令模式下输入`:e ++enc=utf - 8`，这条命令会以`UTF - 8`编码重新加载文件内容。
4. 最后在`vim`的命令模式下输入`:wq`来保存文件并退出`vim`
sudo yum clean all 清理yum缓存
**sudo yum makecache**更新yum仓库信息，该命令会下载所有仓库的元数据，并生成完整的缓存。这意味着它会下载每个仓库的 `repomd.xml` 文件以及相关的元数据文件，如 `primary.xml.gz`、`filelists.xml.gz` 和 `other.xml.gz` 等。
耗时较长。应用场景：适用于需要完整元数据缓存的场景，例如当你需要使用 `yum deplist` 或 `yum repoquery` 等命令查询软件包的依赖关系或详细信息时。
**sudo yum makecache fast**快速缓存生成：该命令只下载仓库的 `repomd.xml` 文件和 `primary.xml.gz` 文件，并生成最小的缓存。primary.xml.gz文件包含了软件包的基本信息，如名称、版本、依赖关系等，但不包含文件列表和其他额外信息。
**耗时较短**，适用于日常使用 YUM 进行软件包安装、更新和查询等操作的场景。对于大多数常规操作，如 `yum install`、`yum update`、`yum list` 等，最小的缓存已经足够。

cat /etc/redhat-release 查看centos系统版本
###### 4.更新yum软件包索引
yum makecache fast
###### 5.安装Docker相关的 docker-ce社区版  ee企业版
```
sudo yum install docker-ce docker-ce-cli containerd.io
```
###### 6.启动Docker
```
sudo systemctl start docker
```
###### 7.查看docker是否安装成功
docker version
docker --version这个命令只显示版本号
######
###### 8.测试docker hello-world
sudo docker run hello-world
后面跳出Hello from Docker!表示Docker安装成功了
![[Pasted image 20241219154539.png]]
![[Pasted image 20241219162244.png]]
###### 9.查看一下下载的这个hello-world镜像
docker images
![[Pasted image 20241219155456.png]]
###### 了解卸载docker
yum remove docker-ce docker-ce-cli containerd.io 卸载依赖
rm -rf /var/lib/docker 卸载资源
##### /var/lib/docker 是docker的默认工作路劲！
##### 镜像加速


---
##### 0.将 `yum` 配置中的源更改为 CentOS Vault
因为CentOS 7 已经停止更新，其官方镜像源已不再提供最新的软件包和更新。
（1）备份现有的YUM配置文件
```
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```
（2）编辑YUM配置文件
vim  /etc/yum.repos.d/CentOS-Base.repo
替换复制成下面的内容
```
[base]
name=CentOS-$releasever - Base
baseurl=https://vault.centos.org/$releasever.9.2009/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates
baseurl=https://vault.centos.org/$releasever.9.2009/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=https://vault.centos.org/$releasever.9.2009/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

确保将 `$releasever` 和 `$basearch` 替换为实际的版本号（7）和架构（x86_64）试了一下，没替换也可以

（3）清除YUM缓存
```
yum clean all
```
(4)生成新的元数据缓存，以便YUM使用新的源
```
yum makecache
```
出现错误curl#6 - "Could not resolve host: mirrorlist.centos.org; 未知的错误"然后进行下面手动安装

##### 1.手动添加Docker官方仓库
sudo nano /etc/yum.repos.d/docker-ce.repo
添加
```
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
```
##### 2.更新YUM缓存
```
sudo yum makecache fast
```
##### 3.安装Docker CE
```
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

![[Pasted image 20241219150533.png]]尝试了添加 Docker 的官方仓库并更新了缓存，但 `yum` 仍然无法找到 `docker-ce`、`docker-ce-cli` 和 `containerd.io` 这些软件包,检查仓库是否正确添加
```
cat /etc/yum.repos.d/docker-ce.repo
```
显示没有这个文件或目录


##### 4.启动Docker服务
启动 Docker 服务   并设置开机自启：
```
sudo systemctl start docker
sudo systemctl enable docker
```

##### 5.验证Docker安装
```
sudo docker run hello-world
```
出现Unable to find image 'hello-world:latest' locally表示 Docker 正在从 Docker Hub 拉取这个镜像。