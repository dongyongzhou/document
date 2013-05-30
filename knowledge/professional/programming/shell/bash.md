---
layout: master
title: shell bash script
---

## Reference


## 如何获取命令的返回值

在Linux系统下察看状态，键入

> $ echo $?

在Windows系统下察看状态，键入

> echo %ERRORLEVEL%

获取pipeline 命令的结果

PIPESTATUS is an array that holds the exit status of your last foreground pipeline commands.

> echo ${PIPESTATUS[0]}


## 如何处理文件Scan

bash循环读入文件的每一行并处理


方法一:
	
	#!/bin/bash
	while read line
	do
		echo $line
	done < urfile
 

方法二:
	
	
	#!/bin/bash
	
	cat urfile | while read line
	do
	    echo $line
	done

## 如何处理输入参数

### 手工处理方式

参数处理-Shell传入参数的处理

	1. $# 传递到脚本的参数个数
	2. $* 以一个单字符串显示所有向脚本传递的参数。与位置变量不同，此选项参数可超过9个
	3. $$ 脚本运行的当前进程ID号
	4. $! 后台运行的最后一个进程的进程ID号
	5. $@ 与$#相同，但是使用时加引号，并在引号中返回每个参数
	6. $- 显示shell使用的当前选项，与set命令功能相同
	7. $? 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。

变量 含义 

	$0 脚本名字 
	$1 位置参数 #1 
	$2 - $9 位置参数 #2 - #9 
	${10} 位置参数 #10 
	$# 位置参数的个数 
	"$*" 所有的位置参数(作为单个字符串) * 
	"$@" 所有的位置参数(每个都作为独立的字符串) 
	${#*} 传递到脚本中的命令行参数的个数 
	${#@} 传递到脚本中的命令行参数的个数 
	$? 返回值 
	$$ 脚本的进程ID(PID) 
	$- 传递到脚本中的标志(使用set) 
	$_ 之前命令的最后一个参数 
	$! 运行在后台的最后一个作业的进程ID(PID)

### getopts/getopt

#### 长选项**--xxx**

	长选项，即选项本身多于一个字符，它也需要一个参数，用等号连接，当然等号不是必须的，参数可以直接写在--xxx，即--xxx参数

getopt


#### 选项**-x**

	选项，它可需要一个参数，也可不需要参数

getopts

		#test.sh
		
		#!/bin/bash
		
		while getopts "a:bc" arg #选项后面的冒号表示该选项需要参数
		do
		        case $arg in
		             a)
		                echo "a's arg:$OPTARG" #参数存在$OPTARG中
		                ;;
		             b)
		                echo "b"
		                ;;
		             c)
		                echo "c"
		                ;;
		             ?) #当有不认识的选项的时候arg为?
		            echo "unkonw argument"
		        exit 1
		        ;;
		        esac
		done



## 文件操作



