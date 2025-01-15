##### 启动Hadoop
 $ cd /usr/local/hadoop
 $ ./sbin/start-all.sh![[Pasted image 20241104151846.png]]
##### Hadoop的三种shell命令
 hadoop fs   适用于任何不同的文件系统，比如本地文件系
 HDFS文件系统
hadoop dfs只能适用于HDFS文件系统
hdfs dfs跟hadoop dfs的命令作用一样，也只能适用于HDFS文件系统
##### Hadoop hdfs的常用命令
1.-help
![[Pasted image 20241104153222.png]]
```
-appendToFile <localsrc> ... <dst>从本地追加信息到HDFS文件系统上
-ls(r) <path> 显示当前目录下所有文件
-du(s) <path> 显示目录中所有文件大小
-count[-q] <path> 显示目录中文件数量
-mv <src> <dst> 移动多个文件到目标目录
-cp <src> <dst> 复制多个文件到目标目录
-rm(r) 删除文件(夹)
-rmdir                   删除空文件夹
-put <localsrc> <dst> 本地文件复制到hdfs
-copyFromLocal <localsrc> ... <dst> 同put
-moveFromLocal 从本地文件移动到hdfs，本地的文件将被删除
-get <src> <localdst> 复制文件到本地，可以忽略crc校验
-getmerge <src> <localdst> 将源目录中的所有文件排序合并到一个文件中
-cat <src>     在终端显示文件内容
-text <src> 在终端显示文件内容
-copyToLocal [-ignoreCrc] <src> <localdst> 复制到本地
-moveToLocal <src> <localdst> 移动到本地，暂未
-mkdir <path> 创建文件夹
-touchz <path> 创建一个空文件
```
2.查看某个指令的详细用法
![[Pasted image 20241104153508.png]]
3.目录操作
第一次使用HDFS时，需要首先在HDFS中创建用户目录。
$hadoop fs -mkdir -p /user/lqh425
显示对应目录下的内容
$hadoop fs -ls /user/lqh425
4.常用操作
查看HDFS的基本使用情况 $hdfs dfsadmin -report
5.进入安全模式，在安全模式下无法进行HDFS的增删改操作
$hdfs dfsadmin -safemode enter
退出安全模式$hdfs dfsadmin -safemode leave
6.创建多级文件夹
一层一层创建$hadoop fs -mkdir /目录一
$hadoop fs -mkdir /目录一/目录二
也可以直接创建$hadoop fs -mkdir -p /目录一/目录二
-p参数的作用是确保如果上级目录不存在，那么Hadoop会自动创建这个上级目录，以及所有必要的父目录，直到创建到指定的目录。
7.查询根目录下的内容
```
$hadoop fs -ls /
```
![[Pasted image 20241105084531.png]]
8.文件夹重命名
$hadoop fs -mv 原目录 目的目录
![[Pasted image 20241105085051.png]]
9.删除文件夹
使用-rm -r或者-rmdir（必须是空的文件夹）命令删除一个目录，使用-rm删除文件。
![[Pasted image 20241105085312.png]]
10.将本地文件上传到HDFS
创建文件nano 文件名
将本地文件上传到HDFS   $hadoop fs -put 本地文件目录 HDFS的目录（-put可以用-copyFromLocal 或-moveFromLocal替换）
使用ls命令查看一下文件是否成功上传到HDFS中$hadoop fs -ls HDFS的目录
查看文件内容$hadoop fs -cat 文件目录
![[Pasted image 20241105085806.png]]
11.将HDFS的文件下载到本地文件系统中
$hadoop fs -get HDFS文件目录 存放到本地文件系统目录
![[Pasted image 20241105090739.png]]12.把HDFS上的一个目录拷贝到另一个目录
$hadoop fs -cp 要拷贝的目录 目的目录
![[Pasted image 20241105091316.png]]
13.查看HDFS文件存储路径Ø （根据每个人配置的时候，选定的datanode路径进行查询）
$cd /usr/local/hadoop/tmp/dfs/data/current/BP-1499990666-192.168.1.101-1726926336256/current/finalized/subdir0/subdir0
![[Pasted image 20241105092121.png]]
查看HDFS在磁盘存储文件内容
$cat 块名
![[Pasted image 20241105093652.png]]14.块拼接
$cat 块名>>新文件名
$sudo tar -zxvf 新文件名
就可以看到旧文件夹的解压
![[Pasted image 20241105094626.png]]![[Pasted image 20241105120042.png]]
15.集群常用端口
![[Pasted image 20241105120204.png]]

##### 编写Hadoop集群常用脚本
1.**H****adoop集群启停脚本（包含HDFS，****Y****arn，Historyserver）：myha****doop.sh**
```
[root@hadoop102 bin]# vim myhadoop.sh
输入如下内容
#!/bin/bash
if [ $# -lt 1 ]
then
  echo "No Args Input..."
    exit ;
fi
case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="
        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/usr/local/hadoop/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/usr/local/hadoop/sbin/start-yarn.sh"
         echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/usr/local/hadoop/sbin/start-yarn.sh"
 # echo " --------------- 启动 historyserver ---------------"
       # ssh hadoop102 "/usr/local/hadoop/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="
        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac                                                  
保存后退出，然后赋予脚本执行权限
[root@hadoop102 bin]# chmod +x myhadoop.sh
```

2.查看三台服务器Java进程脚本：jpsall
```
[root@hadoop102 bin]# vim jpsall
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
        echo =============== $host ===============
        ssh $host source /etc/profile;jps
Done
保存后退出，然后赋予脚本执行权限
[root@hadoop102 bin]# chmod +x jpsall
![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml12296\wps1.jpg)
```
3.分发/home/atguigu/bin目录，保证自定义脚本在三台机器上都可以使用
`[root@hadoop102 bin]# xsync /usr/local/hadoop/bin
4.xsync集群分发脚本
（1）需求：循环复制文件到所有节点的相同目录下
（2）需求分析：（a）scp：需要多次执行(b）期望脚本：
xsync要同步的文件名称
（c）期望脚本在任何路径都能使用（脚本放在声明了全局环境变量的路径）
```
[atguigu@hadoop102 ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin:/opt/module/jdk1.8.0_212/bin```
(3)脚本实现
（a）在/home/atguigu/bin目录下创建xsync文件
[root@hadoop102 bin]# vim xsync
在该文件中编写如下代码
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)
                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
（b）修改脚本 xsync 具有执行权限
[root@hadoop102 bin]# chmod +x xsync
（c）测试脚本
[root@hadoop102 bin]# xsync /usr/local/hadoop/bin
将新增的对应文档都传输到集群的机器当中。