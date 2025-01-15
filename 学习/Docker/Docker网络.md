清空所有环境,即删除所有镜像
```
docker rmi -f $(docker images -aq)
```
docker ps看一下，有运行的就docker stop 镜像id

![[Pasted image 20250110160548.png]]
lo是本地回环地址
eth0是阿里云内网地址
docker0是docker0地址

问题：docker是如何处理容器网络访问的？
docker run -d -P --name tomcat01 tomcat
查看容器的内部网络ip addr,发现容器启动的时候会得到一个eth0@if262 IP地址，是docker分配的，我的电脑没有 因为这个是阿里云收费项目
liunx可以ping通docker容器内部
**原理**
1.我们每启动一个docker容器，docker就会给docker容器分配一个ip,我们只要安装了docker，就会有一个网卡docker0, 桥接模式，使用的技术的evth-pair技术
2.再启动一个容器，会发现又多了一对网卡（ip addr) 18 19 20 21 22 23 
![[Pasted image 20250110171326.png]]
容器带来网卡，都是一对对的
evth-pair就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
正因为有这个特性，evth-pair充当一个桥梁，连接各种虚拟网络设备
OpenStac,Docker容器之间的连接，OVS的连接，都是使用evth-pair技术
3.测试tomcat04和tomcat05是否可以ping通
docker exec -it tomcat05 ping (docker exec -it tomcat04 ip addr里面的etho的地址)
结论：容器和容器之间是可以互相ping通的

![[Pasted image 20250110173340.png]]
结论：tomcat01和tomcat02是公用的一个路由器，docker0
所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用IP


小结：
Docker使用的是Liunx的桥接，宿主机中是一个Docker容器的网桥docker0
Docker中的所有网络接口都是虚拟的，虚拟的转发效率高（就好比内网传递文件）
只要容器删除，对应网桥的一对对就没有了
docker rm -f tomcat05    
ip addr
![[Pasted image 20250110174010.png]]