---
layout:     post
title:      CI持续集成系列之（三）Jenkins的邮件配置及插件使用
subtitle:   在CentOS7.2上配置jenkins
date:       2017-06-22
author:     DY
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - CI持续集成
    - Jenkins
    - 自动部署
---
# 前言

在上一篇我们已经安装配置好了Jenkins,本章主要介绍一些Jenkins入门级别的配置，已经安装一些简单的插件去关联gitlab来拉去代码运行一个job.

# 具体操作
## 配置邮件通知
点击 `系统管理` ==> `系统设置`
![](http://upload-images.jianshu.io/upload_images/3149961-c54da1b479503a2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

找到`Jenkins Location`来添加管理员邮箱。此处的邮箱主要是用来发邮件的
![](http://upload-images.jianshu.io/upload_images/3149961-ad7bc6ce05929c09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置收邮件的地址
![](http://upload-images.jianshu.io/upload_images/3149961-ff66766d691def04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：上面的passwd字段不是qq密码，而是stmp的授权码

## 常用插件安装
点击`系统管理` ==> `管理插件` ==> `直接安装`
![](http://upload-images.jianshu.io/upload_images/3149961-70afeea6bdf49426.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- GitLab Plugin 是用来从gitlab上拉去代码的
- Gitlab Hook Plugin 是用来出发jenkins自动构建插件

PS:如果无法直接安装，可以下载对应的插件放在jenkins机器的目录下/var/lib/jenkins/plugins下即可。插件下载地址：http://updates.jenkins-ci.org/download/plugins/.

## 添加用户认证，用于拉去gitlab代码
![](http://upload-images.jianshu.io/upload_images/3149961-f2ea36075c6a5d0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：这里的rex用户不是gitlab服务的用户，而是gitlab的一个账号。所以他的密码是1我们在gitlab web页面创建的用户密码[123abcdef],而不是linux 系统用户rex密码[111]


在Jenkins服务器上操作生成公约 ,拷贝jenkins的公钥到gitlab服务器 
```
# ssh-keygen  -t rsa 
# cat /root/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCoh18szTmBa+OEU+wNCy+VOKgoW4+kbs1BHoJxsmFELQh+xNijFNypguPwoMU7rvGixwh+zLgipVIKFbD1duCnmj6bXHuBD9rnfJQ2aiAIatB5dxbWkj0vv3gaRU/qx+P8i1Ba6yVqhUMeUj/Uc7OWvV1wmf4bmNdJazVES6N2AAr1iSVtKb2Siy/aL5AO0mZSyooMPuPhe7K+b/oMOEEoolrBpK+yz4Uh5HTGCG3drqEEWe+YtUhQmLTU2F7LJqyPNjXouR42UsbEvyE21QCzmH+YGuY7bx3ThlgGlTtrfxPy6g/J7q/9464/ozpBgw6XXR1z7q3cQ7aGdlVxqN9z root@centos7-2
```
上面jenkins的公约已经有了，我们需要把这拷贝到gitlab服务器上，具体操作如下：

登录gitlab界面，点击右上角选择`Settings` ,点击`SSH Keys`
![](http://upload-images.jianshu.io/upload_images/3149961-518175c9a9ba45fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 快速创建一个一个项目并配置gitlab去拉代码
![](http://upload-images.jianshu.io/upload_images/3149961-d7a80941282dd5fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3149961-d731d717340b166c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
需要注意的是jenkins机器上需要有git命令

![](http://upload-images.jianshu.io/upload_images/3149961-7952a958d5311fb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，test项目完成
