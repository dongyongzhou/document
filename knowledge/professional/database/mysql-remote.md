---
layout: master
title: mysql remote
---

##1 Overview

##2 服务器端配置

使能某一帐户远程访问

###2.2 打开3306端口

使用nestat命令查看3306端口状态：

    $ netstat -an | grep 3306
    tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN

可以看出3306端口只是在IP 127.0.0.1上监听，所以拒绝了其他IP的访问。

解决方法：修改/etc/mysql/my.cnf文件。打开文件，找到下面内容：

    # Instead of skip-networking the default is now to listen only on
    # localhost which is more compatible and is not less secure.
    bind-address  = 127.0.0.1

把上面这一行注释掉或者把127.0.0.1换成合适的IP，建议注释掉。

重新启动后，重新使用netstat检测：

    $ netstat -an | grep 3306
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN

###2.2 修改mysql数据库user用户表

默认是localhost访问。只要在localhost的那台电脑，登入mysql后，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，从"localhost"改称"%"

    mysql -u root -pxxx
    mysql>use mysql; 
    mysql>select host, user from user;
    mysql>update user set host = '%' where user = 'username'; 
    mysql>select host, user from user;

###2.3 授权

**2.3.1 myuser使用mypassword从任何主机连接到mysql服务器**

    mysql>GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WI 
          TH GRANT OPTION; 
    mysql>FLUSH PRIVILEGES;

**2.3.2 允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器，并使用mypassword作为密码**

    mysql>GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 
          'mypassword' WITH GRANT OPTION; 
    mysql>FLUSH PRIVILEGES;

**2.3.3 允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器的dk数据库，并使用mypassword作为密码**

    mysql>GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 
          'mypassword' WITH GRANT OPTION; 
    mysql>FLUSH PRIVILEGES;

GRANT ALL PRIVILEGES ON crashlogdb.* TO 'crashlogdb_user'@'10.231.203.29' IDENTIFIED BY 'crashlogdb_user1' WITH GRANT OPTION; 
10.231.203.29

##3 应用端配置

###3.1 远程命令连接

    $ mysql -h <host ip> -u username -p

###3.2 For ROR

    production:
      adapter: mysql2
      encoding: utf8
      reconnect: false
      host: hostname/IP
      port: portnu
      database: databasename
      pool: 5
      username: username
      password: userpasswd


##4 Reference

* [net-ldap-0.2.2 Documentation](http://net-ldap.rubyforge.org/)
* [Net::LDAP Quick-start for the Impatient](http://net-ldap.rubyforge.org/Net/LDAP.html#method-c-new)
