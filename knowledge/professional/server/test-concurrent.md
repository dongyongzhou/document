---
layout: master
title: Concurrent testing
---

## 目的

测试Web服务器承载的最大并发连接数

测试Web服务器承载的最佳并发数

## 方案

## 测试工具Httperf 

[项目主页](http://code.google.com/p/httperf)

### 介绍

Httperf is a tool for measuring web server performance. It provides a flexible facility for generating various HTTP workloads and for measuring server performance.

The focus of httperf is not on implementing one particular benchmark but on providing a robust, 
high-performance tool that facilitates the construction of both micro- and macro-level benchmarks. 
The three distinguishing characteristics of httperf are 

- its robustness, which  the ability to generate and sustain server overload, 
- support for the HTTP/1.1 and SSL protocols, 
- its extensibility to new workload generators and performance measurements.

### 安装

[下载Httperf](http://code.google.com/p/httperf/downloads/list)

    $ tar xvf httperf-0.9.0.tar.gz
    $ cd httperf-0.9.0/
    $ ./configure
    $ make
    $ sudo make install

## 测试工具autobench

[项目主页](http://www.xenoclast.org/autobench/)

### 介绍

Autobench is a simple Perl script for automating the process of benchmarking a web server (or for conducting a comparative test of two different web servers). The script is a **wrapper around httperf**. Autobench runs httperf a number of times against each host, increasing the number of requested connections per second on each iteration, and extracts the significant data from the httperf output, delivering a CSV or TSV format file which can be imported directly into a spreadsheet for analysis/graphing.

### 安装

[下载autobench](http://www.xenoclast.org/autobench/downloads/)

    $ tar xvf autobench-2.1.2.tar.gz
    $ cd autobench-2.1.2/
    $ make
    $ sudo make install

## 测试

autobench --single_host --host1=192.168.8.8 --port1=80 --uri1=/logo.gif --quiet --low_rate=50 --high_rate=1500 --rate_step=50 --num_call=1 --num_conn=2000 --timeout=10 --file/tmp/result.tsv

autobench --single_host --host1 testhost.foo.com --uri1 /index.html --quiet --file bench.tsv
Benchmark testhost.foo.com using the URI /index.html, don't display httperf output on STDOUT, and save the results in 'bench.tsv'.
autobench --host1 test1.foo.com --host2 test2.foo.com --uri1 /10k.txt --uri2 /10k.txt --rate_step 50 --quiet
Conduct a comparative test of test1.foo.com and test2.foo.com, increasing the requested number of connections per second by 50 on each iteration. Output will go to STDOUT. 

autobench --single_host --host1 www.test.com --uri1 /10K --quiet --low_rate 20 --high_rate 200 --rate_step 20 --num_call 10 --num_conn 5000 --timeout 5 --file results.tsv 

http://www.xenoclast.org/autobench/downloads/