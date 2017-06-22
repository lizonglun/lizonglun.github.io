---
layout:     post
title:      CI持续集成系列之（二）Jenkins安装
subtitle:   在CentOS7.2上安装Jenkins
date:       2017-06-22
author:     DY
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - CI持续集成
    - Jenkins
    - 自动部署
---
# jenkins简介

Jenkins 是一个用 Java 编写的开源的持续Continuous 集成Integration工具。

Jenkins 是用 Java 开发的（界面和 Eclipse一样，带着一股浓浓的 SWT 的味道，好在界面并不太影响使用。），对 Java 程序开发有天然的良好支持，如 JUnit/TestNG 测试，Maven、Ant 等 Java 开发中常用的工具都包含在 Jenkins 里。当然，Jenkins 也可以通过插件来实现其它语言的开发。

在我们公司，Jenkins 主要被用来用于：
```
构建Build、测试Test、部署Deploy代码；
```

我们可以通过一个 Job 实现以下流程：
```
使用 Git 插件，从代码库下载任一版本或分支的源代码；
编译代码；
运行测试。
```

# 搭建流程
## 基础环境准备
```
#系统信息
# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
# uname -r
3.10.0-327.el7.x86_64
```
## 安装JDK
jenkins是java编写的，所以需要先安装JDK环境，这里采用yum安装。
```
# yum install -y java-1.8.0
```
## 安装jenkins
```
# cd /etc/yum.repos.d/
# wget http://pkg.jenkins.io/redhat/jenkins.repo
# rpm --import http://pkg.jenkins.io/redhat/jenkins.io.key
# yum install -y jenkins
```

当然也可以指定源来安装，如下：
```
yum install jenkins --enablerepo=jenkins
```

PS:同样可以通过RPM包指定版本安装： RPM包地址：http://pkg.jenkins-ci.org/redhat/

## 启动服务
```
# systemctl  start jenkins
```

## 登录验证
在浏览器输入：`http://IP:8080`来范文jenkins，为了安全，首先需要解锁Jenkins,请在/var/lib/jenkins/secrets/initialAdminPassword中查看文件。
```
# cat /var/lib/jenkins/secrets/initialAdminPassword
2a3369fd4d384d5e97b468c20222dab1
```
![](http://upload-images.jianshu.io/upload_images/3149961-f02d0e3e47948835.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 安装插件
安装插件呢这合理我们使用建议安装1就行，以后也可以安装的。
![](http://upload-images.jianshu.io/upload_images/3149961-86954c68e9b6c99f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 设置密码
![](http://upload-images.jianshu.io/upload_images/3149961-8853641153739f81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 搭建成功
![](http://upload-images.jianshu.io/upload_images/3149961-d404eef7410e29ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

