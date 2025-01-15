1.准备镜像文件tomcat压缩包，jdk的压缩包！
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.22/bin/apache-tomcat-9.0.22.tar.gz
jdk从学习通上下载
虚拟机进入tomcat目录/usr/local/tomcat
vim Dockerfile
![[Pasted image 20241226200824.png]]
2.编写dockerfile文件，官方命令Dockerfile,build会自动寻找这个文件，就不需要-f指定了
```
FROM centos
MAINTAINER lqh<2769260170@qq.com>
ADD jdk-8u162-linux-x64.tar.gz /usr/local/tomcat
RUN yum -y install vim
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENV JAVA_HOME /usr/local/tomcat/jdk1.8.0_l62
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/tomcat/tomcat9
ENV CATALINA_BASH /usr/local/tomcat/tomcat9
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
EXPOSE 8080
CMD /usr/local/tomcat/tomcat9/bin/startup.sh && tail -F /usr/local/tomcat/tomcat9/bin/logs/catalina.out
```
3.构建镜像
docker build -t diytomcat .
![[Pasted image 20250110153329.png]]

4.启动镜像
docker images
docker run -d -p 9090:8080 --name lqhtomcat -v /tomcat/test:/usr/local/tomcat/tomcat9/webapps/test -v /tomcat/tomcatlogs/:/usr/local/tomcat/tomcat9/logs diytomcat

docker exec -it 镜像id /bin/bash
5.访问测试
在虚拟机
curl localhost:9090 可以看到东西
在浏览器http://39.105.61.80:9090/ 也可以看到东西

6.发布项目（由于做了卷挂载，我们直接在本地编写项目就可以发布了）
在虚拟机tomcat/test目录下
mkdir WEB-INF
cd WEB-INF/
vim web.xml（这里练习是在网上搜索粘贴进来的）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    
    
</web-app>
```
cd ..
vim index.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html> 
<head>
<meta charset="utf-8">
<title>hello. lqh</title>
</head>
<body>
hello,world<br/>
%</body>
%</html>
```
去浏览器上http://39.105.61.80:9090/test/
可以看到内容
cd ..
cd tomcatlogs/
ll
cat catalina.out
