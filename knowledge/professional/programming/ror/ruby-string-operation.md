---
layout: master
title: ruby-string-operation
---

## Overview


## 访问字符串

使用String 的[]方法访问字符串子串

### 取第N个字符

- str[N]：返回它的十进制字符编码
- str[N].chr：返回转换字符编码为实际字符

example：

    actual_name = "log_2012-07-22_00-26-09.txt"
    puts actual_name[2]
    puts actual_name[2].chr

result

    103
    g

### 截取子串

- str[startN,endN]：返回它的startN至endN之间的字符
- str[startN,N]：返回它的startN开始N个字符

example：

    actual_name = "log_2012-07-22_00-26-09.txt"
    puts actual_name[actual_name.length-4,actual_name.length-1]
    puts actual_name[actual_name.length-4,4]

result

    .txt
    .txt

## 字符串比较

### 结束字符串比较

使用正式表达式

先截取结尾字尾串

- file_name[/#{type}$/]：询问file_name结尾是否为#{type}，如果没错就返回#{type}，否则返回nul
- file_name[/^#{type}/]：询问file_name开头是否为#{type}，如果没错就返回#{type}，否则返回nul

- file_name[/#{type}$/，N]：询问file_name结尾从索引N开始是否为#{type}，如果没错就返回#{type}，否则返回nul
- file_name[/^#{type}/，N]：询问file_name开头从索引N开始是否为#{type}，如果没错就返回#{type}，否则返回nul


    def file_sanitization(file_name)
        whilelist=[".txt",".zip"]
        whilelist.each do |type|
                puts type
                puts file_name[/#{type}$/]
                if !file_name[/#{type}$/].nil?
                        puts "GG sanitization testing, pass"
                        return true
                end
        end
        return false
    end

