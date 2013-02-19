---
layout: master
title: Android DropBoxManager Service
---


##1 Reference

* [介绍 Android DropBoxManager Service](http://xiaocong.github.com/blog/2012/11/21/to-introduce-android-dropboxmanager-service/)
* [Android Official Site](http://www.android.com/)
* [DropBoxManager Overview](http://developer.android.com/reference/android/os/DropBoxManager.html)
* [ActivityManager Service Overview](http://developer.android.com/reference/android/app/ActivityManager.html)
* [Android StrictMode Overview](http://developer.android.com/reference/android/os/StrictMode.html)

##2 什么是 DropBoxManager ?

Enqueues chunks of data (from various sources – application crashes, kernel log records, etc.). The queue is size bounded and will drop old data if the enqueued data exceeds the maximum size. You can think of this as a persistent, system-wide, blob-oriented “logcat”.

DropBoxManager 是 Android 在 Froyo(API level 8) 引入的用来持续化存储系统数据的机制, 主要用于记录 Android 运行过程中, 内核, 系统进程, 用户进程等出现严重问题时的 log, 可以认为这是一个可持续存储的系统级别的 logcat.

我们可以通过用参数 DROPBOX_SERVICE 调用 getSystemService(String) 来获得这个服务, 并查询出所有存储在 DropBoxManager 里的系统错误记录.

##3 Android 缺省能记录哪些系统错误 ?

###3.1 crash (应用程序强制关闭, Force Close)

当Java层遇到未被 catch 的例外时, ActivityManagerService 会记录一次 crash 到 DropBoxManager中, 并弹出 Force Close 对话框提示用户.

###3.2 anr (应用程序没响应, Application Not Responding, ANR)

当应用程序的主线程(UI线程)长时间未能得到响应时, ActivityManagerService 会记录一次 **anr** 到 DropBoxManager中, 并弹出 Application Not Responding 对话框提示用户.

###3.3 wtf (What a Terrible Failure)

‘android.util.Log’ 类提供了静态的 wtf 函数, 应用程序可以在代码中用来主动报告一个不应当发生的情况. 依赖于系统设置, 这个函数会通过 ActivityManagerService 增加一个 **wtf** 记录到 DropBoxManager中, 并/或终止当前应用程序进程.

###3.4 strict_mode (StrictMode Violation)

StrictMode (严格模式), 顾名思义, 就是在比正常模式检测得更严格, 通常用来监测不应当在主线程执行的网络, 文件等操作. 任何 StrictMode 违例都会被 ActivityManagerService 在 DropBoxManager 中记录为一次 **strict_mode** 违例.

###3.5 lowmem (低内存)

在内存不足的时候, Android 会终止后台应用程序来释放内存, 但如果没有后台应用程序可被释放时, ActivityManagerService 就会在 DropBoxManager 中记录一次 **lowmem**.


###3.6 watchdog

如果 WatchDog 监测到系统进程(system_server)出现问题, 会增加一条 **watchdog** 记录到 DropBoxManager 中, 并终止系统进程的执行.


###3.7 netstats_error

NetworkStatsService 负责收集并持久化存储网络状态的统计数据, 当遇到明显的网络状态错误时, 它会增加一条 **netstats_error** 记录到 DropBoxManager.

###3.8 BATTERY_DISCHARGE_INFO

BatteryService 负责检测充电状态, 并更新手机电池信息. 当遇到明显的 discharge 事件, 它会增加一条 **BATTERY_DISCHARGE_INFO** 记录到 DropBoxManager.


###3.9 系统服务(System Serve)启动完成后的检测

系统服务(System Serve)启动完成后会进行一系列自检, 包括:

####3.9.1 开机

每次开机都会增加一条 **SYSTEM_BOOT** 记录.

####3.9.2 System Server 重启

如果系统服务(System Server)不是开机后的第一次启动, 会增加一条 **SYSTEM_RESTART** 记录, 正常情况下系统服务(System Server)在一次开机中只会启动一次, 启动第二次就意味着 bug.

####3.9.3 Kernel Panic (内核错误)

发生 Kernel Panic 时, Kernel 会记录一些 log 信息到文件系统, 因为 Kernel 已经挂掉了, 当然这时不可能有其他机会来记录错误信息了. 唯一能检测 Kernel Panic 的办法就是在手机启动后检查这些 log 文件是否存在, 如果存在则意味着上一次手机是因为 Kernel Panic 而宕机, 并记录这些日志到 DropBoxManager 中. DropBoxManager 记录 TAG 名称和对应的文件名分别是:

	SYSTEM_LAST_KMSG, 如果 /proc/last_kmsg 存在.
	APANIC_CONSOLE, 如果 /data/dontpanic/apanic_console 存在.
	APANIC_THREADS, 如果 /data/dontpanic/apanic_threads 存在.

####3.9.4 系统恢复(System Recovery)

通过检测文件 /cache/recovery/log 是否存在来检测设备是否因为系统恢复而重启, 并增加一条 SYSTEM_RECOVERY_LOG 记录到 DropBoxManager 中.

####3.9.5 SYSTEM_TOMBSTONE (Native 进程的崩溃/ANR)

Tombstone 是 Android 用来记录 native 进程崩溃的 core dump 日志, 系统服务在启动完成后会增加一个 Observer 来侦测 tombstone 日志文件的变化, 每当生成新的 tombstone 文件, 就会增加一条 SYSTEM_TOMBSTONE 记录到 DropBoxManager 中.


##4 DropBoxManager 如何存储记录数据 ?

DropBoxManager 使用的是文件存储, 所有的记录都存储在 /data/system/dropbox 目录中, 一条记录就是一个文件, 当文本文件的尺寸超过文件系统的最小区块尺寸后, DropBoxManager 还会自动压缩该文件, 通常文件名以调用 DropBoxManager 的 TAG 参数开头.

	SYSTEM_BOOT@315964824861.txt
	SYSTEM_RESTART@315967836473.txt
	
	event_data@315973829850.txt
	event_log@315964824862.txt
	
	system_app_anr@315968234875.txt
	system_app_crash@315964824868.txt
	system_app_strictmode@315964824864.txt.gz
	system_app_wtf@315967840202.txt
	system_server_watchdog@315967826722.txt.gz
	
	system_app_strictmode
	data_app_strictmode

##5 如何利用 DropBoxManager ?

###5.1 利用 DropBoxManager 来记录需要持久化存储的错误日志信息

DropBoxManager 提供了 logcat 之外的另外一种错误日志记录机制, 程序可以在出错的时候自动将相关信息记录到 DropBoxManager 中. 相对于 logcat, DropBoxManager 更适合于程序的自动抓错, 避免人为因素而产生的错误遗漏. 并且 DropBoxManager 是 Android 系统的公开服务, 相对于很多私有实现, 出现兼容性问题的几率会大大降低.

###5.2 错误自动上报

可以将 DropBoxManager 和设备的 BugReport 结合起来, 实现自动上报错误到服务器. 每当生成新的记录, DropBoxManager 就会广播一个 **DropBoxManager.ACTION_DROPBOX\_ENTRY\_ADDED** Intent, 设备的 BugReport 服务需要侦听这个 Intent, 然后触发错误的自动上报.


