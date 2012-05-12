---
layout: master
title: Concurrent testing
---

##1 目的

测试Web服务器承载的最大并发连接数

测试Web服务器承载的最佳并发数

##2 方案

Httperf + autobench + gnuplot 

##3 测试工具Httperf 

* [项目主页](http://www.hpl.hp.com/research/linux/httperf/)
* [项目 on google code](http://code.google.com/p/httperf)

###3.1 介绍

Httperf is a tool for measuring web server performance. It provides a flexible facility for generating various HTTP workloads and for measuring server performance.

The focus of httperf is not on implementing one particular benchmark but on providing a robust, 
high-performance tool that facilitates the construction of both micro- and macro-level benchmarks. 
The three distinguishing characteristics of httperf are 

- its robustness, which  the ability to generate and sustain server overload, 
- support for the HTTP/1.1 and SSL protocols, 
- its extensibility to new workload generators and performance measurements.

###3.2 安装


* [下载Httperf: FTP](ftp://ftp.hpl.hp.com/pub/httperf/)
* [下载Httperf: Sourceforge](http://sourceforge.net/projects/httperf/)
* [下载Httperf: GoogleCode](http://code.google.com/p/httperf/downloads/list)

    $ tar xvf httperf-0.9.0.tar.gz
    $ cd httperf-0.9.0/
    $ ./configure
    $ make
    $ sudo make install

###3.3 Usage

[Httperf Doc](http://www.hpl.hp.com/research/linux/httperf/wisp98/httperf.pdf)

##4 测试工具autobench

[项目主页](http://www.xenoclast.org/autobench/)

###4.1 介绍

Autobench is a simple Perl script for automating the process of benchmarking a web server (or for conducting a comparative test of two different web servers). The script is a **wrapper around httperf**. Autobench runs httperf a number of times against each host, increasing the number of requested connections per second on each iteration, and extracts the significant data from the httperf output, delivering a CSV or TSV format file which can be imported directly into a spreadsheet for analysis/graphing.

###4.2 安装

[下载autobench](http://www.xenoclast.org/autobench/downloads/)

    $ tar xvf autobench-2.1.2.tar.gz
    $ cd autobench-2.1.2/
    $ make
    $ sudo make install

###4.3 usage&configuration

**Configuration File**

Since autobench takes a large number of different options, which, in practice, typically won't vary a great deal, it allows you to specify defaults for the majority of options in a configuration file $HOME/.autobench.conf. The first time you run autobench, it will merely install this configuration file and exit. Thereafter it will run normally. Any options specified on the command line override those in the configuration file.

##5 gnuplot

Included with Autobench 1.1.0 (and later) is a program to generate graphs with **gnuplot** from an Autobench results file. See the bench2graph man page for details.

[项目主页](http://www.gnuplot.info/)
###5.1 安装

    sudo apt-get install gnuplot
    $ gnuplot -V
    gnuplot 4.4 patchlevel 2

###5.2 问题及解决

**"set data style" not supported**

    $ bench2graph results.tsv result.png
    
    Enter the title : 
    
    set data style linespoints
     ^
    "gnuplot.cmd", line 6: Unrecognized option.  See 'help set'.

bench2graph

原因：4.4版本不支持 set data style

解决办法：

    cp /usr/local/bin/bench2graph /usr/local/bin/bench2png（根本不必要）

    sed -i 's/postscript color/png xffffff/g' /usr/local/bin/bench2png

或 

    找到：echo set terminal postscript color > gnuplot.cmd
    改为：echo set terminal png large size 640,480 truecolor > gnuplot.cmd

    找到：echo set data style linespoints >> gnuplot.cmd
    改为：echo set style data linespoints >> gnuplot.cmd

##6 测试

###6.1 benchmark a single server

    autobench --single_host --host1=192.168.8.8 --port1=80 --uri1=/logo.gif --quiet --low_rate=50 --high_rate=1500 --rate_step=50 --num_call=1 --num_conn=2000 --timeout=10 --file/tmp/result.tsv

    autobench --single_host --host1 xxx.com --uri1 /index.html --quiet --file bench.tsv

Benchmark testhost.foo.com using the URI /index.html, don't display httperf output on STDOUT, and save the results in 'bench.tsv'.

###6.2 conduct tests against two machines

    autobench --host1 test1.foo.com --host2 test2.foo.com --uri1 /10k.txt --uri2 /10k.txt --rate_step 50 --quiet

Conduct a comparative test of test1.foo.com and test2.foo.com, increasing the requested number of connections per second by 50 on each iteration. Output will go to STDOUT. 

    autobench --single_host --host1 www.test.com --uri1 /10K --quiet --low_rate 20 --high_rate 200 --rate_step 20 --num_call 10 --num_conn 5000 --timeout 5 --file results.tsv 

##7 结果分析

###7.1 Display the analysis Graphs

    $ bench2graph result.tsv result.png [1 2 3 ... n]

( 1 2 3 ... n: 表示 Results.tsv 结果集文件中的项目 )


默认全部显示,共10项

1 dem_req_rate    
2 req_rate_xx.com    
3 con_rate_xx.com    
4 min_rep_rate_xx.com        
5 avg_rep_rate_xx.com        
6 max_rep_rate_xx.com        
7 stddev_rep_rate_xx.com     
8 resp_time_xx.com   
9 net_io_xx.com      
10 errors_xx.com

可以多项选择显示如

    $ bench2graph result.tsv result.png 1 3 7 9


