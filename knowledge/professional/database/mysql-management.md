---
layout: master
title: mysql management
---

## MYSQL服务控制

### ubuntu mysql启动｜停止｜重启
 
    $ sudo /etc/init.d/mysql start  
    $ sudo /etc/init.d/mysql stop  
    $ sudo /etc/init.d/mysql restart 

### 连接与断开服务器

    $ mysql -h host -u user -p

## MYSQL数据库管理

### MySQL用户账户管理

###

### 1. 查询Mysql数据库大小

** mathod one **

    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

SHOW TABLE STATUS的性质与SHOW TABLE类似，不过，可以提供每个 表的大量信息。您也可以使用mysqlshow --statusdb_name命 令得到此清单。

** mathod two **

- 进入information_schema 数据库（存放了其他的数据库的信息）

    use information_schema;

- 查询所有数据的大小：

    select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;
 

- 查看指定数据库的大小：

比如查看数据库home的大小

    select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';
 

- 查看指定数据库的某个表的大小

比如查看数据库home中 members 表的大小

    select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';

## Reference

* [MySQL 5.1参考手册](http://dev.mysql.com/doc/refman/5.1/zh/index.html)