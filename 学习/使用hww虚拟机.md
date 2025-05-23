<<<<<<< HEAD
hadoop100虚拟机，密码000000

##### 安装的软件有
anaconda      hadoop-3.3.5    pycharm-community-2022.2.3        jdk-8u371-linux-x64
kafka_2.11-0.10.2.0.                spark-3.4.0-bin-without-hadoop
mysql-connector-java-5.1.40     
在pyspark环境下，安装了flask   flask-socketio  kafka-python   pyarrow flask-login werkzeug

=======
>>>>>>> 6cbad1b6ead4534cd6d01757fb15acc07912bcd0
启动pycharm
```
cd /usr/local/pycharm
./bin/pycharm.sh
<<<<<<< HEAD
```

查看虚拟环境的那些下载路径
```
python
import sys
print(sys.path)
```

pyspark虚拟环境的是
```
/home/atguigu/anaconda3/envs/pyspark/lib/python3.8/site-packages
```
主环境base的虚拟环境是
```
/home/atguigu/anaconda3/lib/python3.11/site-packages
```

|​**​安装方式​**​|​**​安装位置​**​|​**​主环境访问​**​|​**​虚拟环境访问​**​|
|---|---|---|---|
|虚拟环境内 `pip install`|虚拟环境的 `site-packages`|❌ 不可用|✅ 可用（仅当前虚拟环境）|
|系统级 `sudo pip install`|`/usr/local/lib/pythonX.X/site-packages`|✅ 可用|❌ 默认不可用（除非使用 `--system-site-packages`）|
|非 Python 软件（如 Kafka）|`/usr/local/bin` 或自定义路径|✅ 可用|✅ 可用（通过绝对路径或 `$PATH` 配置）|
通过命令，可以知道该路径下已经安装了
```
ls -l /usr/local/spark/jars | grep "spark-streaming"
```
![[Pasted image 20250513224923.png]]
我们这里只有一个这个文档是因为，现在已经把它们合并成一个东西。
我们这里是已经弄好了spark开发kafka环境了的
=======
```
>>>>>>> 6cbad1b6ead4534cd6d01757fb15acc07912bcd0
