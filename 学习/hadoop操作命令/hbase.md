法一：进入zookeeper:hbase zkcli
![[Pasted image 20241110225618.png]]
法二：内置zookeepr
输入jps显示有HQuorumPeer，它表示hbase管理的zookeeper
![[Pasted image 20241119091618.png]]
```
ps -ef | grep HQuorumPeer 
```
用来找到`HQuorumPeer` 的实际路径，进而找到 `zkCli.sh`
看到 `HQuorumPeer` 进程是由 Java 启动的，环境位于 `/usr/lib/jvm/jdk/bin/java`
```
ls /usr/local/hbase/lib/ | grep zookeeper
```
在 HBase 的 `lib` 目录下查找是否有 Zookeeper 的客户端工具
看到`/usr/local/hbase/lib/` 目录下有 `zookeeper-3.4.10.jar`，这是一个 Zookeeper 的 Java 库文件，但它并不是 Zookeeper 客户端工具 `zkCli.sh`。这个 JAR 文件是 HBase 用来与 Zookeeper 通信的客户端库，并不包含命令行工具。
```
echo "stat" | nc localhost 2181
```
获取zookeeper的状态
```
echo "ls /" | nc localhost 2181
```
列出节点的子节点
```
echo "ruok" | nc localhost 2181
```
检查 Zookeeper 服务器是否处于健康状态。如果 Zookeeper 服务运行正常，会收到 `imok` 作为响应。
![[Pasted image 20241119091959.png]]
法三：输入hbase zkcli

![[Pasted image 20241122165154.png]]
要是命令行那里0变成1，输入end再回车