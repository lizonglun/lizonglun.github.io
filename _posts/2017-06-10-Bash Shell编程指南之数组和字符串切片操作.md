---
layout:     post
title:      Bash Shell编程指南之字符串切片
subtitle:   
date:       2017-06-10
author:     DY
header-img: img/post-bg-char.jpg
catalog: true
tags:
    - Bash
    - 数组
---
# 数组
## 数组的定义
变量是存储单个元素的内存空间，那么数组是存储多个连续的内存空间。这个数组只有一个名字，且索引编号从0开始。
> 注意：bash-4及之后的版本，支持自定义索引格式，而不仅仅是0，1,2，...数字格式；此类数组称之为“**关联数组**”

## 数组的声明
```
declare  -a  NAME：声明索引数组；
declare  -A  NAME：声明关联数组；
```
## 数组元素的赋值方式
1.一次只赋值一个元素：`ARRAY_NAME[INDEX]=value`
```
# name[0]=rex
# name[1]=tom
# echo ${name[*]}
rex tom

```
2.一次赋值全部元素：`ARRAY_NAME=("VAL1"  "VAL2"  "VAL3"  ...)`
```
# mytest=(one two three four)
# echo ${mytest[*]}
one two three four
```
3.只赋值特定的元素`ARRAY_NAME=([0]="VAL1"  [3]="VAL4" ...)`
```
# name=([3]="jerry" [5]="john")
# echo ${name[3]}
jerry
# echo ${name[5]}
john
```
> 注意：bash支持稀疏格式的数组；(所谓稀疏格式即数组中的元素未必全部都存在，某个位置元素数值可以为空)

4.read  -a  ARRAY_NAME
```
[root@rex ~]# read -a name
rex tom jerry
[root@rex ~]# echo ${name[\*]}
rex tom jerry
```
## 向非稀疏格式的数组中追加元素
```
[root@rex ~]# week[${#week[\*]}]=haha
[root@rex ~]# echo ${week[\*]}
Mon Tue Wed Thu Fri Sat Sun haha

```

## 数组的引用`${ARRAY_NAME[INDEX]}`
1.引用所有数组
```
# echo ${mytest[\*]}
# ${mytest[@]}
```
2.引用单个数组
```
# echo ${name[0]}

```
> 注意：引用时，只给数组名，表示引用下标为0的元素；环境变量数组的索引值都是从零开始

3.显示数组中元素的个数：`${#ARRAY_NAME[\*]}`
```
[root@rex ~]# name=(tom john jerry rex)
[root@rex ~]# echo ${name[\*]}
tom john jerry rex
[root@rex ~]# echo ${#name[\*]}
4
```
4.显示数组内元素的长度`${#ARRAY_NAME[#]}`
```
[root@rex ~]# echo ${#name[2]} //#代表数字
```
## 删除数组中的元素`unset  ARRAY[INDEX]`
```
[root@rex ~]# echo ${week[\*]}
Tue Wed Thu Fri Sat Sun haha

[root@rex ~]# unset week[${#week[\*]}]
[root@rex ~]# echo ${week[\*]}
Tue Wed Thu Fri Sat Sun

```
## 数组元素切片

语法格式：`${ARRAY_NAME[@]:offset:number}`

- offset：要路过的元素
- number：要去取出的元素个数，省略是，表示取偏移量之后的所有元素

范例：取元素操作
```
[root@rex ~]# week=(Mon Tue Wed Thu Fri Sat Sun)
[root@rex ~]# echo ${week[\*]}
Mon Tue Wed Thu Fri Sat Sun
[root@rex ~]# echo ${week[@]:2:2}   
Wed Thu
[root@rex ~]# echo ${week[@]:2}
Wed Thu Fri Sat Sun
```
## 关联数组
bash-4及之后的版本，支持自定义索引格式，而不仅仅是0，1,2，...数字格式；此类数组称之为关联数组
```
declare  -A  ARRAY_NAME
ARRAY_NAME=([index_name1]="value1"  [index_name2]="value2" ...)
```
# 字符串操作
## 字符串切片
语法格式：`${var:offset:number}`
- 取字符串子串（负索引技术法）
```
[root@rex ~]# name=jerry
[root@rex ~]# echo ${name: -4} // -4前有一个空格
erry

```
## 基于模式取子串
语法格式：`${var#*word}`
- 其中word是指定的分隔符
功能：自左而右，查找var变量所存储的字符串中，**第一次出现的word分隔符**，删除字符串开头至此分隔符之间的所有字符；
```
[root@rex ~]# name=123ab456
[root@rex ~]# echo ${name#*ab}
456
```
语法格式：`${var##*word}`：其中word是指定的分隔符；
功能：自左而右，查找var变量所存储的字符串中，**最后一次出现的word分隔符**，删除字符串开头至此分隔符之间的所有字符；
```
mypath="/etc/init.d/functions"
${mypath##*/}:   取得结果：functions - 以“/”为分隔符，删除第一次出现“/”到最后一次出现“/”的字符
${mypath#*/}:  取得结果： etc/init.d/functions - 以“/”为分隔符，删除第一次出现“/”之前的内容
```		
语法格式：`${var%word*}`：其中word是指定的分隔符；
功能：自右而左，查找var变量所存储的字符串中，最后一次出现的word分隔符，删除此分隔符至字符串尾部之间的所有字符；
```
mypath="etc/init.d/functions"
${mypath%/*}:  取得结果： etc/init.d
${mypath%%/*}:  取得结果：etc
```
```
[root@rex ~]# url=http://poinetech.com:90
[root@rex ~]# echo ${url##*:}
90
[root@rex ~]# echo ${url%%:*}
http
```
## 字符串的查找替换
`${var/PATTERN/SUBSTI}`：查找var所表示的字符串中，第一次被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；
`${var//PATTERN/SUBSTI}`：查找var所表示的字符串中，所有被PATTERN所匹配到的字符串，并将其全部替换为SUBSTI所表示的字符串；
`${var/#PATTERN/SUBSTI}`：查找var所表示的字符串中，行首被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；如果不是行首，不予替换；
`${var/%PATTERN/SUBSTI}`：查找var所表示的字符串中，行尾被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；如果不是行尾，不予替换；

> 注意：这里的PATTERN中使用glob风格和通配符；

## 字符串的查找删除
`${var/PATTERN}`：以PATTERN为模式查找var字符串中第一次的匹配，并删除之；
`${var//PATERN}`：。。。。。。。。。。。。。。。。所有的。。。。。。。。
`${var/#PATTERN}` 。。。。。。。。。。。。。。。。。行首。。。。。。。。
`${var/%PATTERN}` 。。。。。。。。。。。。。。。。。行尾。。。。。。。。

## 字符串大小写转换
`${var^^}`：把var中的所有小写字符转换为大写；
`${var,,}`：把var中的所有大写字符转换为小写；

