[TOC]

## mysql基础操作

+ 创建database
create database hive default charset utf8 collate utf8_general_ci;

+ 添加用户
create user 'username@host' [IDENTIFIED BY 'PASSWORD']

+ 通过grant授权
grant all privileges on  database.tablename to username@ip identified by 'passwd';


