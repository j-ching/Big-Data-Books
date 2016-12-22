title: hue 安装和使用
notebook: 技术相关
tags: hue

[TOC]

# hue

### hue安装准备

+ 安装 python-devel

```
yum install python-devel.x86_64

```

+ 安装 mysql

```
yum install mysql-server

```

### hue workflow中文乱码问题
+ ALTER TABLE beeswax_savedquery MODIFY COLUMN data varchar(500) character set utf8;
+ ALTER TABLE beeswax_savedquery MODIFY COLUMN `desc` longtext  character set utf8;
+ ALTER TABLE desktop_document2 MODIFY COLUMN description longtext  character set utf8;
+ 有些中文字符址导入的时候无法正确识别，目前的解决方法是通过修改标的字符集啦i 解决，lodadata的时候应该会有参数可以指定字符转换。Cd /home/hadoop/huehue loaddata a.json  