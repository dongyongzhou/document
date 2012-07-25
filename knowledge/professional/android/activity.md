---
layout: master
title: Android Activity
---

# Activity

## Activity Overview

## Activity Lifecycle

## Activity and Threads

### Android APK应用程序有哪些线程呢？

至少包含三个线程

- Binder Thread： ViewRoot.W
- Binder Thread： ApplicationThread对象。

两者都负责接收Linux Binder驱动发送的IPC调用

- UI Thread，处理用户消息，绘制界面


### Android应用自定义thread and UI thread区别

- UI Thread从ActivityThread运行的，已添加Looper对象，即已经为该线程创建了消息队列。可以Activity中定义handler对象，可以接受发送的消息。
- 自定义thread：是一个祼线程，不能直接在Thread中定义Handler对象，即不能给Thread对象发消息。

## Activity class view

![Activity 类图](http://hi.csdn.net/attachment/201106/27/62017_1309154560MBjJ.png)

frameworks/base/core/java/android/app

    ActivityGroup.java
    Activity.java  
    ActivityManager.java  
    ActivityManagerNative.java  
    ActivityThread.java

### ActivityThread:

ActivityThread主要用来启动应用程序的主线程，并且管理在应用端跟用户打交道的activity。在应用端的activity信息全部被存储在ActivityThread的成员变量mActivities中。

    final HashMap<IBinder, ActivityClientRecord> mActivities
            = new HashMap<IBinder, ActivityClientRecord>();

在mActivities中，记录了应用程序创建的所有activity实例记录，对应的是ActivityRecord。

ActivityThread是怎么启动应用程序的呢？

ActivityThread中有一个main函数，在这个里面，将启动应用程序并建立消息循环。系统会为主线程自动创建消息循环。

    public static void main(String[] args) {
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();
        if (sMainThreadHandler == null) {
            sMainThreadHandler = new Handler();
        }

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

在建立消息循环之前，会通过thread.attach(false)来初始化应用程序的运行环境，并建立activityThread和ActivityManagerService之间的桥mAppThread， mAppThread是IApplicationThread的一个实例。

    private void attach(boolean system) {
        sThreadLocal.set(this);
        mSystemThread = system;
        if (!system) {
            ViewRootImpl.addFirstDrawHandler(new Runnable() {
                public void run() {
                    ensureJitEnabled();
                }
            });
            android.ddm.DdmHandleAppName.setAppName("<pre-initialized>");
            RuntimeInit.setApplicationObject(mAppThread.asBinder());
            IActivityManager mgr = ActivityManagerNative.getDefault();
            try {
                mgr.attachApplication(mAppThread);
            } catch (RemoteException ex) {
                // Ignore
            }


每个应用程序对应着一个ActivityThread实例，应用程序由ActivityThread.main打开消息循环。每个应用程序同时也对应着一个ApplicationThread对象。该对象是activityThread和ActivityManagerService之间的桥梁。

在attach中还做了一件事情，就是通过代理调用attachApplication，并利用binder的transact机制，在ActivityManagerService中建立了ProcessRecord信息。

之后通过该ProcessRecord就可以获得该ActivityThread中的所有ActivityRecord记录。

![](http://hi.csdn.net/attachment/201106/27/62017_1309154561sT75.png)


### ActivityManagerService

在ActivityManagerService中，也有一个用来管理activity的地方：mHistory栈，这个mHistory栈里存放的是服务端的activity记录HistoryActivity（class HistoryRecord extendsIApplicationToken.Stub）。处于栈顶的就是当前running状态的activity。

Activity的startActivity方法的请求过程：

![](http://hi.csdn.net/attachment/201106/27/62017_1309154561aAz8.png)

从该时序图中可以看出，Activity.startActivity()方法最终是通过代理类和Binder机制，在ActivityManagerService.startActivity方法中执行的。

那么在ActivityManagerService的startActivity中，主要做了那些事情？

    public final int startActivity(IApplicationThread caller,
            Intent intent, String resolvedType, Uri[] grantedUriPermissions,
            int grantedMode, IBinder resultTo,
            String resultWho, int requestCode, boolean onlyIfNeeded, boolean debug,
            String profileFile, ParcelFileDescriptor profileFd, boolean autoStopProfiler) {
        return mMainStack.startActivityMayWait(caller, -1, intent, resolvedType,
                grantedUriPermissions, grantedMode, resultTo, resultWho,
                requestCode, onlyIfNeeded, debug, profileFile, profileFd, autoStopProfiler,
                null, null);
    }


activity被加入到mHistory之后，只是说明在服务端可以找到该activity记录了，但是在客户端目前还没有该activity记录。还需要通过ProcessRecord中的thread（IApplication）变量，调用它的scheduleLaunchActivity方法在ActivityThread中创建新的ActivityRecord记录。


![](http://hi.csdn.net/attachment/201106/27/62017_130915456111WI.png)


ApplicationThread中的scheduleLaunchActivity方法, 在这个里面主要是根据服务端返回回来的信息创建客户端activity记录ActivityRecord. 并通过Handler发送消息到消息队列，进入消息循环。在ActivityThread.handleMessage()中处理消息。最终在handleLaunchActivity方法中把ActivityRecord记录加入到mActivities（mActivities.put(r.token,r)）中，并启动activity


- 在客户端和服务端分别有一个管理activity的地方，服务端是在mHistory中，处于mHistory栈顶的就是当前处于running状态的activity，客户端是在mActivities中。

- 在startActivity时，首先会在ActivityManagerService中建立HistoryRecord，并加入到mHistory中，然后通过scheduleLaunchActivity在客户端创建ActivityRecord记录并加入到mActivities中。最终在ActivityThread发起请求，进入消息循环，完成activity的启动和窗口的管理等

### how a new process is created?


## ActivityManager

An activity’s state is managed by the runtime’s ActivityManager


## Activity Manager Service

### AMS 主要功能

- 统一调度各应用程序的Activity。应用程序运行activity->先报告AmS->AmS决定是否可能启动,通知应用程序运行指定的Activity。
- 内存管理。Activity退出时，其进程不会被杀死，只有在内存紧张时才由AmS杀死，
- 进程序管理。向外提供查询系统正在运行的进程信息API。

ActivityManagerService提供了一个ArrayList mHistory来管理所有的activity，activity在AMS中的形式是ActivityRecord，task在AMS中的形式为TaskRecord，进程在AMS中的管理形式为ProcessRecord。如下图所示

![](http://hi.csdn.net/attachment/201112/25/0_1324812260QNUm.gif)

A **task** is a collection of activities that users interact with when performing a certain job. The activities are arranged in a stack (the "back stack"), in the order in which each activity is opened.

[Tasks and Back Stack](http://developer.android.com/guide/topics/fundamentals/tasks-and-back-stack.html)

### 进程数据类ProcessRecord

frameworks/base/services/java/com/android/server/am

记录一个进程中的相关信息

- 进程文件信息
- 进信息内存信息
- 进程包含的Activity、provider、Service等。

### HistoryRecord

保存每个Activity的信息

- 环境信息
- 运行状态信息

### TaskRecord

AmS使用任务的概念确保Activity启动和退出的顺序。
