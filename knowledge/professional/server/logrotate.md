---
layout: master
title: logrotate
---

##1 Overview

logrotate 程序是一个日志文件管理工具。用来把旧的日志文件删除，并创建新的日志文件，我们把它叫做“转储”。我们可以根据日志文件的大小，也可以根据其天数来转储，这个过程一般通过 cron 程序来执行。logrotate 程序还可以用于压缩日志文件，以及发送日志到指定的E-mail 。

日志文件的管理：

1. logrotate 配置
2. 缺省配置 logrotate
3. 使用 include 选项读取其他配置文件
4. 使用 include 选项覆盖缺省配置
5. 为指定的文件配置转储参数

##2 logrotate 配置

logrotate 的配置文件是 /etc/logrotate.conf

$ man logrotate

 参数                     功能

- compress                通过gzip 压缩转储以后的日志
- nocompress              不需要压缩时，用这个参数
- copytruncate            用于还在打开中的日志文件，把当前日志备份并截断
- nocopytruncate          备份日志文件但是不截断
- create mode owner group 转储文件，使用指定的文件模式创建新的日志文件
- nocreate                不建立新的日志文件
- delaycompress           和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
- nodelaycompress         覆盖 delaycompress 选项，转储同时压缩。
- errors address          专储时的错误信息发送到指定的Email 地址
- ifempty                 即使是空文件也转储，这个是 logrotate 的缺省选项。
- notifempty              如果是空文件的话，不转储
- mail address            把转储的日志文件发送到指定的E-mail 地址
- missingok               如果日志不存在则忽略该警告信息
- nomail                  转储时不发送日志文件
- olddir directory        转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
- noolddir                转储后的日志文件和当前日志文件放在同一个目录下
- prerotate/endscript     在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
- postrotate/endscript    在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行
- daily                   指定转储周期为每天
- weekly                  指定转储周期为每周
- monthly                 指定转储周期为每月
- rotate count            指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
- tabootext [+] list      不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig .rpmsave 
- size                    当日志文件到达指定的大小时才转储，可以指定bytes(缺省)以及K或者M
- dateext                 使用日期作为命名格式
- dateformat .%s          配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，
                          必须配合dateext使用，只支持 %Y %m %d %s 这四个参数

##3 缺省配置 logrotate

/etc/logrotate.conf


    # rotate log files weekly
    weekly

    # keep 4 weeks worth of backlogs
    rotate 4

    # create new (empty) log files after rotating old ones
    create

    # uncomment this if you want your log files compressed
    #compress

    # packages drop log rotation information into this directory
    include /etc/logrotate.d


##4 使用 include 选项读取其他配置文件

    # packages drop log rotation information into this directory
    include /etc/logrotate.d

就是从/etc/logrotate.d里读取配置文件的配置信息。允许不同软件使用不同的配置方式。

    $ ls  /etc/logrotate.d
    apache2  apport  apt  aptitude  dpkg  mysql-server  ppp  rsyslog  samba  ufw  unattended-upgrades  vsftpd  winbind
c
其中可以看到apache2的配置文件

##5 使用 include 选项覆盖缺省配置

可以将一些类似logrotate.conf的配置信息加到这个目录，覆盖旧的配置。

##6 为指定的文件配置转储参数

对特定日志文件使用配置信息如下。

    /full/path/to/file{
         option(s)
    }

类似apache2的配置文件

     /var/log/apache2/*.log {
        weekly
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 root adm
        sharedscripts
        postrotate
                /etc/init.d/apache2 reload > /dev/null
        endscript
    }
 
prerotate 命令指定转储以前的动作

postrotate 指定转储后的动作

endscript 结束 prerotate、postrotate 部分的脚本

尽管花括号的开头可以和其他文本放在同一行上，但是结尾的花括号必须单独成行。
