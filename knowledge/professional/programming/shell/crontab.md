---
layout: master
title: crontab
---

## Reference

引自：http://baike.baidu.com/view/1229061.htm




[例行性工作排程 (crontab)](http://linux.vbird.org/linux_basic/0430cron.php)

## Overview

crontab命令常见于Unix和类Unix的操作系统之中，用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。该词来源于希腊语 chronos(χρνο)，原意是时间。

通常，crontab储存的指令被守护进程激活， crond常常在后台运行，每一分钟检查是否有预定的作业需要执行。这类作业一般称为cron jobs。

## Setting

To edit the crobjob, It need to execute:

	$sudo crontab -u xxx -e


	[root@www ~]# crontab [-u username] [-l|-e|-r]
	選項與參數：
	-u  ：只有 root 才能進行這個任務，亦即幫其他使用者建立/移除 crontab 工作排程；
	-e  ：編輯 crontab 的工作內容
	-l  ：查閱 crontab 的工作內容
	-r  ：移除所有的 crontab 的工作內容，若僅要移除一項，請用 -e 去編輯。
	
	範例一：用 dmtsai 的身份在每天的 12:00 發信給自己
	[dmtsai@www ~]$ crontab -e
	# 此時會進入 vi 的編輯畫面讓您編輯工作！注意到，每項工作都是一行。
	0   12  *  *  * mail dmtsai -s "at 12:00" < /home/dmtsai/.bashrc
	#分 時 日 月 週 |<==============指令串========================>|

## 查看

	
	該如何查詢使用者目前的 crontab 內容呢？我們可以這樣來看看：
	
	[dmtsai@www ~]$ crontab -l
	59 23 1 5 * mail k	iki < /home/dmtsai/lover.txt
	*/5 * * * * /home/dmtsai/test.sh
	30 16 * * 5 mail friend@his.server.name < /home/dmtsai/friend.txt
	
	# 注意，若僅想要移除一項工作而已的話，必須要用 crontab -e 去編輯～
	# 如果想要全部的工作都移除，才使用 crontab -r 喔！
	[dmtsai@www ~]$ crontab -r
	[dmtsai@www ~]$ crontab -l
	no crontab for dmtsai
	


