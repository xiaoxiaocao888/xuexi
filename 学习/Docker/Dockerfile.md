Dockerfile就是用来构建docker镜像的构建文件！命令脚本
玩一下：
1.在虚拟机来到home目录下，mkdir docker-test-volume
2.pwd       显示当前的路径
3.cd docker-test-volume/
4.vim dockerfile1
文件中的内容是(指令要大写)：
```
FROM centos
VOLUME ["volume01","volume02"]
CMD echo "---end---"
CMD /bin/bash
```
通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个一个命令，每个命令都是一层！（匿名挂载）
5.docker build -f /home/docker-test-volume/dockerfile1 -t lqh/centos:1.0 .

-f指的是那个文件，-t要生成的镜像
6.docker images可以看到lqh/centos:1.0
7.启动自己生成的镜像
docker images
docker run -it 镜像id /bin/bash
ls 可以看到我们生成镜像的时候自动挂载的数据卷目录
这个卷和外部一定有一个同步的目录
8.在容器内cd volume01
touch container.txt
ls
9.查看卷挂载的路径
在虚拟机上
docker ps
docker inspect 镜像id
可以看到Mounts，复制volume01的路径

10.测试一下刚才新建的文件是否同步出去了
在虚拟机上
cd 刚刚复制的路径
ls
可以看到container.txt文件
假设构建镜像的时候没有挂载卷，要手动镜像挂载 -v 卷名:容器内路径

##### dockerfile介绍
dockerfile是用来构建docker镜像的文件，命令参数脚本
###### 构建步骤
1.编写一个dockerfile文件
2.docker build 构建成为一个镜像
3.docker run 运行镜像
4.docker push发布镜像（DockerHub、阿里云镜像仓库）
查看官方是怎么做的（开梯子，在hub.docker上搜索软件名，点概述下面就点dockerfile就会跳转）
![[Pasted image 20241226160520.png]]
很多官方的镜像都是基础包，很多功能没有，我们通常会自己搭建自己的镜像

##### dockerfile构建过程
基础知识：
1.每个保留关键字（指令）都是必须是大写字母
2.执行从上到下顺序执行
3.#表示注释
4.每一个指令都会创建提交一个新的镜像层，并提交！
![[Pasted image 20241226170315.png]]
dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单！
Docker镜像逐渐成为企业交付的标准，必须要掌握
步骤：开发，部署，运维。缺一不可
Dockerfile:构建文件，定义了一切的步骤，源代码
DockerImages:通过DockerFile构建生成的镜像，最终发布和运行的产品
Docker容器：容器就是镜像运行起来提供服务器

##### Dockerfile的指令
FROM                     基础镜像，一切从这里开始构建
MAINTAINER           镜像是谁写的，姓名+邮箱
RUN                        镜像构建的时候需要运行的命令
ADD                         步骤：tomcat镜像，这个tomcat压缩包！添加内容
WORKDIR                  镜像的工作目录
VOLUME                    挂载的目录
EXPOSE                     保留端口配置
CMD                         指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENIRYPOINT             指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD                 当构建一个被继承DockerFile 这个时候会运行ONBUILD的指令，触发指令
COPY                        类似ADD，将我们的文件拷贝到镜像中
ENV                           构建的时候设置环境变量

##### 实战测试
Docker Hub中99%镜像都是从这个基础镜像过来的FROM scratch，然后配置需要的软件和配置来进行构建

1.在虚拟机home目录下，mkdir dockerfile
cd dockerfile
vim mydockerfile-centos
```
FROM centos
MAINTAINER lqh<2769260170@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash
```
2.通过这个文件构建镜像
命令docker build -f dockerfile文件路径 -t 镜像名:[tag]
docker build -f mydockerfile-centos -t mycentos:0.1 .
可以看到centos是直接拿来使用的，因为我们已经下载了一个centos镜像了，没有的话才要去下载
docker run -it mycentos:0.1
pwd
3.测试运行
对比之前的原生的centos
工作目录默认是根目录，没有vim、ifconfig等命令

我们增加之后的镜像
默认的工作目录是/usr/local （pwd)
可以使用ifconfig、vim命令

4.我们可以列出本地进行的变更历史
docker history 镜像id
我们平时拿到一个镜像，可以研究一下它是字母做的了

##### CMD和ENIRYPOINT区别
测试cmd
虚拟机编写dockerfile文件，在/home/dockerfile目录下
vim dockerfile-cmd-test
```
FROM centos
CMD ["ls","-a"]
```
构建镜像
docker build -f dockerfile-cmd-test -t cmdtest .
run运行，发现我们的ls -a命令生效
docker run 镜像id
想追加一个命令 -l   ls -al
docker run 镜像id -l
报错了
因为在CMD，-l替换了ls -a命令，而-l不是命令所以报错

测试ENTRYPOINT
vim dockerfile-cmd-entrypoint
```
FROM centos
CMD ["ls","-a"]
```
docker build -f dockerfile-cmd-entrypoint -t entoryponit-test .
docker run 镜像id

我们的追加命令，是直接拼接在我们的ENTRYPOINT命令的后面
docker run 镜像id -l
