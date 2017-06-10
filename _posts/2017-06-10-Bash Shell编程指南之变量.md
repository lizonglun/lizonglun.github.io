---
layout:     post
title:      Bash Shell编程指南之变量
subtitle:   
date:       2017-06-10
author:     DY
header-img: img/post-bg-bash.jpg
catalog: true
tags:
    - Bash
    - shell
---
# 变量基础及深入
## 什么是变量？
变量是一个用固定的字符串（也可能是字符数字等的组合），替代更多更复杂的内容，这个内容里可能还会包含变量和路径，字符串等其他的内容。使用变量最大的好处就是方便，当然，除了方便以外，很多时候在变成中使用变量也是必须的，否则就无法完成开发工作了。
```
# name='hello world'  //定义变量
# echo $name          //调用变量
hello world
```
- **变量类型**
- - `环境变量`：环境变量也称全局变量，可以在创建它们的shell及其派生出来的子shell中使用。
- - `普通变量`：普通变量也称局部变量，只能在当前shell中使用。

## 环境变量
环境变量用于定义shell的运行环境，保证Shell命令的正确执行，通过环境变量来确定登陆用户名、命令路径、终端类型、登陆目录等，所有的环境变量都是系统全局变量，可用于所有子进程中，这包括编辑器、Shell脚本和各类应用。

环境变量可以在命令行中设置，当用户退出时这些变量值也会丢失，因此最好在用户家目录下的`.bash_profile`文件中或全局配置`/etc/bashrc`,`/etc/profile`文件或者`/etc/profile.d/`中定义。将环境变量放入上述的文件中，每次用户登陆时这些变量值都将被初始化。

传统上，所有环境变量均为大写。环境变量应用于用户进程前，都应该用export命令导出，例如：export AS=1

### 查看环境变量
1.env（只显示全局变量）
* .set（所有的变量）
* .declare（所有变量，包括函数、整数和已导出的）

用户环境变量的配置
```
# ll  /root/.bash_profile 
-rw-r--r--. 1 root root 176 Dec 29  2013 /root/.bash_profile
# ll /root/.bashrc 
-rw-r--r--. 1 root root 176 Dec 29  2013 /root/.bashrc
```
全局环境变量的配置
```
# ll /etc/profile
-rw-r--r--. 1 root root 1750 Jun  7  2013 /etc/profile
# ll /etc/bashrc 
-rw-r--r--. 1 root root 2835 Aug 12  2015 /etc/bashrc
```
用户登陆后加载的内容，可以在跳板机上操作，让每切换一次用户就显示当前登陆的USER and IP切换或者登陆用户时提示信息
```
# vim /etc/profile.d/hello.sh
echo $USER
# sudo su -
```
用户第一次登陆时提示，只能是字符串
```
Last login: Sat Jun 10 11:50:43 2017 from 192.168.1.7
hello boys!!!
# cat /etc/motd 
hello boys!!!
```
### 定义全局变量
```
export NAME=rex
NAME=rex
export NAME
export LANG=EN
declare -x NAME=rex
```
### 显示或取消全局变量
设置与取消全局变量
```
$ echo $NAME
ansheng
$ unset NAME
$ echo $NAME
```
系统常见的一些环境变量
```
$ echo $HOME			用户登录时进入的目录
$ echo $UID			当前用户UID（用户标识）相当于id -u
$ echo $PWD			当前工作目录的绝对路径
$ echo $SHELL			当前SHELL
$ echo $USER			当前用户
```
### 定义本地变量
本地变量指的是在用户当前shell中使用，当shell退出，或者启动一个新的shell时是无效的

- 普通字符串变量定义
```
变量名=value
变量名='value'
变量名="value"
命令变量定义
变量名=$()
变量名=``
```
- shell中变量’声明‘的要求
一般是由字母，数字，下划线组成，以字母开头
```
$ a=192.168.1.2
$ b='192.168.1.2'
$ c="192.168.1.2"
$ echo "a=$a"
a=192.168.1.2
$ echo "b=$b"
b=192.168.1.2
$ echo "c=${c}"
c=192.168.1.2
```
```
$ a=192.168.1.2-$a
$ b='192.168.1.2-$a'
$ c="192.168.1.2-$a"
$ echo "a=$a"
a=192.168.1.2-192.168.1.2
$ echo "b=$b"
b=192.168.1.2-$a
$ echo "c=${c}"
c=192.168.1.2-192.168.1.2-192.168.1.2
```

**提示**:
- 第一种定义a变量的方式是直接指定变量内容，内容一般为简单连续的数字、字符串、路径明等。
- 第二种定义b变量的方式是通过单引号定义变量。这个方式会 的特点是：输出变量时引号里是什么就输出什么，即使内容中有变量也会把变量原样输出，此方法比较适合定义显示纯字符串。
- 第三种定义c变量方式是通过双引号定义变量。这个方式的特点是：输出变量时引号里的变量会经过解析后输出该变量内容。而不是把引号中变量名元杨树称呼，适合于字符串中附带有变量的内容的定义。

> `习惯上数字不加引号，其他默认加双引号`

## 变量中单引号、双引号以及不加引号的区别
- **单引号**：强引用，不会被变量替换。
- **双引号**：弱引用，会被变量替换。
- **无引号**：是什么就是什么，会被变量替换，但是如果有特殊符号，空格则输出为空

范例1：
```
[root@rex ~]# NAME=rex
[root@rex ~]# echo $NAME
rex
[root@rex ~]# echo '$NAME'
$NAME
[root@rex ~]# echo "$NAME"
rex
```
范例2：
```
[root@rex ~]# REX=123
[root@rex ~]# awk 'BEGIN {print $REX}'

[root@rex ~]# awk 'BEGIN {print '$REX'}'
123
[root@rex ~]# awk 'BEGIN {print "$REX"}'  //注意这里，使用双引号
$REX
```

### 自定义普通字符串变量的建议
- 内容时纯数字（不带空格），定义方式可以不加引号（单或双）
- 没特殊情况，字符串一般用双引号定义，特别是多个字符串中间有空格时
- 变量内容需要原样输出时，要用单引号（``） 例如：NAME='REX'
### 变量的命名规范
- 变量命名要统一，使用全部大写字母，语句要清晰，能够正确表达变量内容的含义，过长的英文单词可采用前几个自负代替，多个单词连接使用_号链接，引用时，最好以${APACHE_ERR_NUM}加大括号或"${APACHE_ERR_NUM}"外面加双引号方式引用变量。

- 避免无意义字符或数字：例如下面的COUNT，并不知道其确切含义；
范例：CONUT的不确切定义
```
CONUT=`grep keywords file`
if [ ${CONUT} -ne 22 ]
then
	echo “Do Something”
	fi
```

- 全局变量和局部变量命名
- - 脚本中定义全局变量REX，在使用是使用{}来进行引用，例如：{REX}
范例：操作系统函数库脚本群居变量截取示例
```
# cat /etc/init.d/functions 
```
- - 在脚本中定义局部变量
存在于脚本函数（function）中的变量称为局部变量，要以local方式进行声明，使只只在本函数作用域内有效，防止变量在函数中的命名与变量外部程序中变量重名造成异常.

范例：函数中的变量定义

```
checkpid() {
        local i


	for i in $* ;do
	        [ -d "/proc/$i" ] && return 0
		done
		return 1
		}
```

- 变量合并
某些变量或配置项要组合起来才有意义时，如文件的路径和文件名称，建议将要组合的变量合并到一起赋值给一个新的变量，这样既方便之后的调用，也为以后进行修改提供了方便。

范例：自动化安装HTTPD的脚本变量合并定义
```
VERSION="2.2.22"
SOFTWARE_NAME="httpd"
SOFTWARE_FULLNAME="${SOFTWARE}-${VERSION}.tar.gz"
```
## 定义变量总结
多学习模仿操作系统自带的`/etc/init.d/function`函数库脚本的定义思路。

1.变量名只能为字母，数字，下划线，字母开头
* .规范的变量名定义方法：见名只意
* .=号的知识，a=1等号是赋值的意思
* .比较是不是相等，为==打印变量，变量名前接$符号，变量名后面紧接着字符的时候，要用大括号括起来例如：`rex == $NAME`
* .注意变量内容引用方法，一般用引号，简单连续字符串可以不加引号，希望原样输出，使用单引号。
* .变量内容是命令，这个时候要用反引号或者$()把变量括起来使用。例如：NAME=$(hostname)。

## 特殊变量
### 位置变量

符号 | 描述
---- | ---
$0 | 获取当前shell脚本文件名，包括脚本路径
$n | 获取当前shell脚本的第n个参数，`n=1..9`，当`n>10`时，要用大括号${10}进行引用
$\* | 获取当前shell脚本的所有参数，将所有命令行参数视为单个字符串，相当于“$1$2$3”,注意与$#的区别
$# | 获取当前shell脚本命令行中参数的总个数
$@ | 获取程序的所有参数$1 $2 $3 …，这是将参数传递给其他程序的最佳方式，因为他会保留内嵌在每个参数里的任意空白


范例：判断参数的个数，强制要求输入两个参数

```
# cat my.sh 
#!/bin/bash
 [ $# -ne 2 ] && {
 	echo "you need Enter 2 argu"  //必须输入连个参数
	exit
	}
echo $1 $2
```

范例：$\*与$@的区别
- $\*将所有的命令行所有参数视为单个字符串，等同于$1$2$3,$\*
- $@将命令行每个参数视为单独的字符串，等同于$1 $2 $3。这是将参数传递给其他程序的最佳方式，因为他会保留所有内嵌在每个参数里的任何空白
> 上述区别在于双引号的时候，即 $\* 和 $@


```
[root@rex ~]# set -- "I am" good man   //传入3个参数
[root@rex ~]# echo $# 
3
[root@rex ~]# for i in "$*";do echo $i;done    //在有双引号的情况下，"$*"中的所有参数，当成一个参数输出了
I am good man
[root@rex ~]# for i in "$@";do echo $i;done   //在有双引号的情况下，"$@"中的参数每个参数都独立输出了
I am
good
man
[root@rex ~]# 
[root@rex ~]# for i ;do echo $i;done     //去掉in变量列表，相当于in “$@” 
I am
good
man
[root@rex ~]# for i in $*;do echo $i;done   //不加双引号，把所有参数输出，然后第一个参数“I am”也拆开输出了
I
am
good
man
[root@rex ~]# for i in $@;do echo $i;done   //不加双引号，把所有参数输出，然后第一个参数“I am”也拆开输出了
I
am
good
man

```
### 进程的状态变量

符号 | 描述
---- | ---
$$ | 获取当前shell脚本的进程PID
$! | 执行上一个指令的PID
$? | 获取上一个指令的返回值（0：成功，非0：失败）
$_ | 在此之前执行的命令或脚本的最后一个参数

范例： $$案例
```
#!/bin/sh
pidpath=/tmp/a.pid
if [ -f "$pidpath" ];then
        killl -USR2 `cat $pidpath` > /dev/null 2>&1
	rm -rf $pidpath
fi
	echo $$ > $pidpath
	sleep 300
```
范例:$_案例
```
[root@rex ~]# /etc/init.d/network restart
Restarting network (via systemctl):                        [  OK  ]
[root@rex ~]# echo $_
restart
```
## 变量赋值的高级用法
`${var:-VALUE}`：如果var变量为空，或未设置，那么返回VALUE；否则，则返回var变量的值； 
`${var:=VALUE}`：如果var变量为空，或未设置，那么返回VALUE，并将VALUE赋值给var变量；否则，则返回var变量的值； 
`${var:+VALUE}`：如果var变量不空，则返回VALUE；
`${var:?ERROR_INFO}`：如果var变量为空，或未设置，那么返回ERROR_INFO为错误提示；否则，返回var值； 
