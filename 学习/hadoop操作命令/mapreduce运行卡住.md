方法一：在mapred-site.xml文件（好像是hadoop安装目录再下面的/etc/hadoop）将
```
<property>
      <name>mapreduce.framework.name</name>
       <value>yarn</value>
</property>
```
改成
```
<property>
      <name>mapreduce.job.tracker</name>
      <value>hdfs://ip:8001</value>
     <final>true</final>
</property>
```
其中ip 为master具体的IP地址，如192.168.1.129
