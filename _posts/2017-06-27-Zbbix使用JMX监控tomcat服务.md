---
layout:     post
title:      Zabbix3.0使用JMX监控tomcat服务
subtitle:   在CentOS7.2上使用zabbix官方源安装zabbix，监控Tomcat服务
date:       2017-06-26
author:     DY
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - zabbix
    - 监控系统
    - JMX
---
# 前言

上篇我们使用阿里源的仓库监控了一台linux主机的基本资源。本章我们在上篇的基础上监控tomcat服务。

# 监控Tomcat服务的原理

当Zabbix-Server需要知道java应用程序的某项性能的时候，会启动自身的一个Zabbix-JavaPollers进程去连接Zabbix-JavaGateway请求数据，而ZabbixJavagateway收到请求后使用“JMXmanagementAPI”去查询特定的应用程序，而前提是应用程序这端在开启时需要“-Dcom.sun.management.jmxremote”参数来开启JMX远程查询就行。Java程序会启动自身的一个简单的小程序端口12345向Zabbix-JavaGateway提供请求数据。

![](http://www.msbgn.cn/msbgn/tomcat211.png)

从上面的原理图中我们可以看出，配置Zabbix监控Java应用程序的关键点在于：
- 配置Zabbix-JavaGateway
- 让Zabbix-Server能够连接Zabbix-JavaGateway
- Tomcat开启JVM远程监控功能等

下面我们来具体配置

## zabbix-server端

1.安装JDK
```
# rpm -ivh jdk-8u25-linux-x64.rpm 
```
2.配置JDK环境变量，查看设置是否成功
```
# vim /etc/profile.d/java.sh
JAVA_HOME=/usr/java/jdk1.8.0_25
PATH=$PATH:$JAVA_HOME
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
# . /etc/profile.d/java.sh
```
3.安装zabbix-java-gateway
```
# yum install zabbix-java-gateway
# systemctl start zabbix-java-gateway
# ss -tnl|grep 10052
LISTEN     0      50          :::10052                   :::*  
```
4.修改zabbix-java-gateway的配置文件，并重启服务
```
# vim /etc/zabbix/zabbix_java_gateway.conf
LISTEN_IP="0.0.0.0"
LISTEN_PORT=10052
START_POLLERS=5
# systemctl restart zabbix-java-gateway
```
5.修改zabbix-server的配置文件，增加如下内容：
```
# vim /etc/zabbix/zabbix_server.conf
JavaGateway=127.0.0.1
JavaGatewayPort=10052
StartJavaPollers=0
# 注意：Zabbix Server/Proxy中的StartJavaPollers要小于等于Zabbix Java GateWay配置文件中的START_POLLERS
```
5.重启zabbix-server服务
```
# systemctl  restart zabbix-server
```

# zabbix-agent端
1.部署Tomcat应用
```
# rpm -ivh jdk-8u25-linux-x64.rpm 
# tar xf apache-tomcat-7.0.42.tar.gz  -C /usr/local/
# cd /usr/local/
# ln -sv apache-tomcat-7.0.42/ tomcat
```
2.修改catalina.sh文件，开启JMX
```
# vim ./tomcat/bin/catalina.sh
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote #开启远程监控
  -Dcom.sun.management.jmxremote.port=12345 #远程监控端口
  -Dcom.sun.management.jmxremote.ssl=false #远程ssl验证为false
  -Dcom.sun.management.jmxremote.authenticate=false #关闭权限认证
  -Djava.rmi.server.hostname=192.168.157.129" #部署了tomcat的主机地址
```
3.启动重启tomcat服务
```
# ../bin/startup.sh 
```
4.下载catalina-jmx-remote.jar，并且放入 cd /usr/local/tomcatweixin/lib/
下载地址：链接：http://pan.baidu.com/s/1eSKOzNc 密码：f29s
```
# cp catalina-jmx-remote.jar /usr/local/tomcat/lib/
```
5.重启tomcat服务
```
# /usr/local/tomcat/bin/catalina.sh stop
# /usr/local/tomcat/bin/catalina.sh start
```
6.验证是否启用JMX监听成功
```
# lsof -i :10053
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    33812 root   20u  IPv6 100240      0t0  TCP *:10053 (LISTEN)
```
7.导入cmdline-jmxclient-0.10.3.jar包在zabbix-server进行测试
```
# java -jar cmdline-jmxclient-0.10.3.jar  - 192.168.157.129:10053 java.lang:type=Memory NonHeapMemoryUsage

06/26/2017 21:29:23 -0400 org.archive.jmx.Client NonHeapMemoryUsage: 
committed: 22478848
init: 2555904
max: -1
used: 21709768
# 有数据返回说明成功了
```

# 配置zabbix监控页面
1.导入tomcat监控模板，可以自定义模板，也可以使用zabbix自带的模板


# 报错
`Error: Exception thrown by the agent : java.net.MalformedURLException: Local host name unknown: java.net.UnknownHostException: centos7-2: centos7-2: unknown erro`
解决方案：添加hosts文件解析

