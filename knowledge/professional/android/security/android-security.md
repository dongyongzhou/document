---
layout: master
title: about-source-envsetup
---

## 问题

为什么我们编译Android代码时，需要输入：  source ./build/envsetup.sh  或者 . ./build/envsetup.sh哪？ （这里的source和.的作用是一致的）， 为什么不可以直接执行envsetup.sh脚步而需要通过source命令执行哪？


## Linux 环境变量的机制

Linux的环境变量是存储于RAM中的，每个Process启动时，OS会往Process的RAM中写入环境变量，所以每个Process的环境变量间是相互独立的。
Linux每个Process启动时的初始环境变量是从其父进程继承过来的，但是一旦子进程启动后，那么不会再和父进程的环境变量存在任何依赖关系，子进程的环境变量的更改不会影响父进程，反之亦然。

所以，要控制程序运行时能获取的环境变量，只能在父进程中写入。

## 回答

Linux中，标准方式运行Shell Script会导致启动一个新的shell进程来运行Script。 对于envsetup.sh而言，如果以标准方式执行，那么就会启动一个新的shell进程来运行，运行完成退回到当前的shell进程（我们的编译shell环境）。envsetup.sh内部定义了一系列的环境变量和shell函数，期望在我们的编译环境中被使用。那么，如果使用标准方式（非Source）执行时，这些环境变量和函数的定义将只会在新的shell进程（当前编译环境所在shell进程的子进程）中生效，当envsetup.sh执行后返回其父进程（当前编译环境所在shell）时，所有envsetup.sh中定义的环境变量和函数在此编译环境shell中并没有生效，违背了我们的意愿，后续的编译就不能引用了，比如mm，mmm都不能引用。

此时就需要使用source命令，在当然shell中使用source命令执行envsetup.sh时，不会fork出新的shell进程来运行，而是直接在当前shell进程中读取envsetup.sh文件来运行，这样使得envsetup.sh中的环境变量和函数的定义在当前的用户编译环境shell中生效。

## Android/Linux中环境变量的其他一些说明

由于init是User Space的1号进程，所以在init.rc中 Export的环境变量将在所有的User Space进程中可见。Zygote中设置的环境变量，将在所有的Android APK 进程中可见，但是在Native Process中不可见。

Native的环境变量的读写函数是，getenv/setenv

Java层的环境变量的读写函数是：System.getProperties().getProperty/setProperty     注意和Android Property的区别：

System.getproperty/setproperty

另外，由于安全问题（这里不详述，又是另一个话题）所有具有SUID/SGID属性的Linux的可执行文件(包括.so)在运行时，会在自身进程中删除一系列和安全相关的继承来的环境变量，比如LD_LIBRARY_PATH等，使得在其和其子进程中无法继承和访问系统的这些环境变量值。


