---
layout:     post
title:      Elsaticsearch篇
subtitle:   在CentOS7上部署Elasticsearch集群
date:       2017-02-06
author:     DY
header-img: img/2017-06-01-elasticsearch.png 
catalog: true
tags:
    - ELKstack
    - 日记
    - 日志分析
---

# 1.环境配置
主机名解析与时间同步
```
[root@node1 ~]# cat /etc/hosts
192.168.123.111 node1 www.node1.com
192.168.123.112 node2 www.node2.com
192.168.123.113 node3 www.node3.com
192.168.123.114 node4 www.node4.com
```
```
[root@node1 ~]# ntpdate  0.centos.pool.ntp.org
[root@node2 ~]# ntpdate  0.centos.pool.ntp.org
[root@node3 ~]# ntpdate  0.centos.pool.ntp.org
[root@node4 ~]# ntpdate  0.centos.pool.ntp.org
```
# 2.主机实验用途说明
- node1和node2为elasticsearch集群(不部署Logstash) 
- node3收集对象,Nginx、java、tcp、syslog等日志 
- node4将logstash日志写入Redis,减少程序对elasticsearch依赖性，同时实现程序解耦以及架构扩展。 
- 被收集主机需要部署Logstash。
主机名	IP	内存	服务
node1	192.168.123.111	1G	Elasticsearch、Kibana
node2	192.168.123.112	1G	Elasticsearch、Kibana
node3	192.168.123.113	1G	Logstash、服务及程序日志
node4	192.168.123.114	1G	Logstash、Redis(消息队列)
# 一、Elasicsearch集群部署
## 1.安装jdk环境
```
[root@node2 ~]# yum install  java
```
## 2.下载GPG key
```
[root@node2 ~]# rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch 
```
## 3.配置elasticsearch的yum仓库
```
[root@node1 ~]# vim /etc/yum.repos.d/elasticsearch.repo
[sticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```
## 4.安装elasticsearch
```
[root@node1 ~]# yum install  elasticsearch 
[root@node2 ~]# yum install -y elasticsearch
[root@node1 ~]# rpm -q elasticsearch
elasticsearch-2.4.5-1.noarch
```
## 5.配置elasticsearch并启动
```
[root@node1 ~]# mkdir -p /data/es-data
[root@node1 ~]# chown -R elasticsearch.elasticsearch /data/es-data
[root@node1 ~]# vim /etc/elasticsearch/elasticsearch.yml
cluster.name: my-cluster
node.name: node1
path.data: /data/es-data
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 192.168.123.111
http.port: 9200
discovery.zen.ping.unicast.hosts: ["node1", "node2"]  #要是用集群需要有这一项

[root@node1 ~]# systemctl  start elasticsearch.service
[root@node1 ~]# ss -tnl|grep -E "9200|9300"
LISTEN     0      50      ::ffff:192.168.123.111:9200                    :::*                  
LISTEN     0      50      ::ffff:192.168.123.111:9300                    :::*     
```
```
[root@node2 ~]# mkdir -p /data/es-data
[root@node2 ~]# chown -R elasticsearch.elasticsearch /data/es-data
[root@node2 ~]# vim /etc/elasticsearch/elasticsearch.yml
cluster.name: my-cluster
node.name: node2
path.data: /data/es-data
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 192.168.123.112
http.port: 9200
discovery.zen.ping.unicast.hosts: ["node1", "node2"]  #要是用集群需要有这一项

[root@node2 ~]# systemctl  start elasticsearch
[root@node2 ~]# ss -tnl|grep -E  "9300|9200"
LISTEN     0      50      ::ffff:192.168.123.112:9200                    :::*                  
LISTEN     0      50      ::ffff:192.168.123.112:9300                    :::*  
```
## 6.访问elasticsearch_url
```
[root@node2 ~]# curl http://node1:9200
{
  "name" : "node1",
    "cluster_name" : "my-cluster",
    "cluster_uuid" : "hurHQj1RThCSJco5OKjKJQ",
        "version" : {
	 "number" : "2.4.5",
	 "build_hash" : "c849dd13904f53e63e88efc33b2ceeda0b6a1276",
	 "build_timestamp" : "2017-04-24T16:18:17Z",
	 "build_snapshot" : false,
	 "lucene_version" : "5.5.4"
	},
   "tagline" : "You Know, for Search"
}
说明部署没问题了
```
# Elastic插件介绍
```
/usr/share/elasticsearch/bin/plugin -h
```
## a.安装集群管理插件-head插件
```
[root@node1 ~]# /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head
```
访问head插件：·`http://192.168.123.111:9200/_plugin/head`

## b.安装监控管理插件-kopf
```
[root@node1 ~]# /usr/share/elasticsearch/bin/plugin install lmenezes/elasticsearch-kopf
```
访问插件：`http://192.168.123.111:9200/_plugin/kopf`


