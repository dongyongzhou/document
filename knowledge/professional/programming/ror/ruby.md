---
layout: master
title: Ruby
---

# Ruby

## 参考资料

* [Ruby的官方网站](http://www.ruby-lang.org/en/)
* [Programming Ruby](http://www.rubycentral.com/pickaxe/)
* [Ruby On Rails官方网站](http://rubyonrails.org/)
* [Mr. Neighborly’s Humble Little Ruby Book](http://www.humblelittlerubybook.com/book/html/index.html)
* [RailsCast](http://railscasts.com/)
* [The Ruby Toolbox](https://www.ruby-toolbox.com/)
* [Get to the Point!](http://johnwlong.com/slides/gettothepoint/)

## Ruby 

* [File Operation](ruby-file-operation.html)
* [String Operation](ruby-string-operation.html)

### Ruby 变量

Ruby属于动态类型。

在大多数语言里,变量都必须指定其类型,可更改性(是不是个常数)和范围;,在Ruby里变量不需要声明.

>
一般小写字母、底线开头：变量（Variable）。

$开头：全局变量（Global variable）。

@开头：实例变量（Instance variable）。

@@开头：类变量（Class variable）类别变量被共享在整个继承链中

大写字母开头：常数（Constant）。

- 局部变量
  
  小写字母或'_'起始, 局部变量不像全局和实变量一样在初始化前含nil值

  作用域：定义它的方法或循环内部


- 实例变量
 
  @作字首, 像全局变量一样,实变量在初始前的值是nil

  作用域：限制在 self 对象内
  
- 类变量

  @@作字首

- 全局变量

  $作字首  在初始化前,全局变量有一个特殊的值 nil.

  作用域：程序的任何位置
  
- 常量

  大写字母开头  它应最多被赋值一次.

  作用域：常量可以定义在类里,但不像实变量,它们可以在类的外部访问.

## End 
