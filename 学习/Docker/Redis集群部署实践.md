![[Pasted image 20250111115518.png]]
部署redis网络 要创建了网络，再启动节点的哦
```
docker network create redis --subnet 172.38.0.0/16
```
查看redis网络详细信息
```
docker network inspect redis
```
通过脚本创建六个redis配置
```
for port in $(seq 1 6);\
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat <<EOF>/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.con \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \

docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

#创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1

```
docker搭建redis集群完成

我们使用了docker之后，所有的技术都会慢慢的变得简单起来

![[Pasted image 20250111130921.png]]
![[Pasted image 20250111131045.png]]
一个一个启动
![[Pasted image 20250111131148.png]]
![[Pasted image 20250111131239.png]]
![[Pasted image 20250111131416.png]]进入集群redis-cli -c
![[Pasted image 20250111131332.png]]
暂停节点3，测试高可用性，发现节点5变为主节点
![[Pasted image 20250111131418.png]]
![[Pasted image 20250111131351.png]]