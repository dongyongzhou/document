---
layout: master
title: fileObserver
---

## OverView

android.os包下的FileObserver类是一个用于监听文件访问、创建、修改、删除、移动等操作的监听器，基于linux的INotify。

FileObserver是个抽象类，必须继承它才能使用。每个FileObserver对象监听一个单独的文件或者文件夹，如果监视的是一个文件夹，那么文件夹下所有的文件和级联子目录的改变都会触发监听的事件。


## problem


FileObserver对象必须保持一个引用，确保不被垃圾收集器回收掉，否则就不会触发事件，这里可以考虑使用Service服务。

## Reference

[Android文件或文件夹内容改变监听器（FileObserver）](http://blog.csdn.net/mayingcai1987/article/details/6210904)

