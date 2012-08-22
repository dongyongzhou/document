---
layout: master
title: Android Not Response
---

# Overview

In Android, the system guards against applications that are insufficiently responsive for a period of time by displaying a dialog to the user, called the Application Not Responding (ANR) dialog, shown at right in Figure 1. The user can choose to let the application continue, but the user won't appreciate having to act on this dialog every time he or she uses your application. It's critical to design responsiveness into your application, so that the system never has cause to display an ANR dialog to the user.

Generally, the system displays an ANR if an application cannot respond to user input. For example, if an application blocks on some I/O operation (frequently a network access), then the main application thread won't be able to process incoming user input events. After a time, the system concludes that the application is frozen, and displays the ANR to give the user the option to kill it.

ANR就是Application Not Responding的全称，即应用程序无响应。如果某个应用程序有一段时间响应不够灵敏而出现响应超时时，Android系统会弹出一个对话框上面写道，XXX is not responding给出两个按钮一个为force close一个为wait。用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要，这样，系统不会显示ANR给用户。

一般说来，如果应用程序不能响应用户输入的话，系统会显示一个ANR。例如，一个应用程序阻塞在一些I/O操作上（通常是网络访问），这时，应用程序的主线程就不能再处理用户的输入事件。经过一定的时间后，系统认为应用程序已经挂起，并显示ANR来让用户选择杀死应用程序。

## What and Why

在Android里，应用程序的响应性是由Activity Manager和Window Manager系统服务监视的。当它监测到以下情况中的一个时，Android就会针对特定的应用程序显示ANR：

- 在5秒内没有响应输入的事件（例如，按键按下，屏幕触摸） 
(If there is no response to the **input**, e.g., keypress, touch, within 5 sec, WindowManager triggers ANR.)

InputManager kicks out the timer when it dispatches a key event to the UI chain of the foreground process. If it does not respond within time, WindowManager detects and prints out the message with the trace log and kills the process. An alert message also displays on the screen.

超时时间的计数一般是从按键分发给app开始。此类超时的原因一般有两种：

    (1)当前的事件没有机会得到处理（即UI线程正在处理前一个事件，没有及时的完成或者looper被某种原因阻塞住了）
    (2)当前的事件正在处理，但没有及时完成

- BroadcastReceiver在10秒内没有执行完毕
( If **BroadcastReceiver** does not finish executing within 10 sec, ActivityManager triggers ANR.)

- ServiceTimeout(20 seconds) --小概率类型. Service在特定的时间内无法处理完成

Service在20秒内没有执行完毕

Every time there is a broadcast event, e.g., Wi-Fi on/off, GPS on/off, boot complete message, etc., it kicks a timeout before it broadcasts the event to all the processes that had registered a receiver. If there is a process not responding with a complete message back within a given time, ActivityManager kills that process and prints out the log.

## ANR 深入分析

    I/BadBehaviorActivity( 1947): ANR pressed -- about to hang
    I/WindowManager(  432): Input event dispatching timed out sending to com.android.development/com.android.development.BadBehaviorActivity
    E/ActivityManager(  432): ANR in com.android.development (com.android.development/.BadBehaviorActivity)
    E/ActivityManager(  432): Reason: keyDispatchingTimedOut
    F/libc    ( 1947): Fatal signal 6 (SIGABRT) at 0x000001b0 (code=0)
    I/BadBehaviorActivity( 1947): hang finished -- returning
    I/DEBUG   (  167): *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
    I/DEBUG   (  167): Build fingerprint: 'xxxx'
    I/DEBUG   (  167): pid: 1947, tid: 1947  >>> com.android.development <<<
    I/DEBUG   (  167): signal 6 (SIGABRT), code 0 (?), fault addr --------

### 显示定义

./frameworks/base/core/res/res/values-pt/strings.xml

    <!-- Title of the alert when an application is not responding. -->
    <string name="anr_title"></string>
    <!-- Text of the alert that is displayed when an application is not responding. -->
    <string name="anr_activity_application"><xliff:g id="application">%2$s</xliff:g> is not responding.\n\nWould you like to close it?</string>
    <!-- Text of the alert that is displayed when an application is not responding. -->
    <string name="anr_activity_process">Activity <xliff:g id="activity">%1$s</xliff:g> is not responding.\n\nWould you like to close it?</string>
    <!-- Text of the alert that is displayed when an application is not responding. -->
    <string name="anr_application_process"><xliff:g id="application">%1$s</xliff:g> is not responding. Would you like to close it?</string>
    <!-- Text of the alert that is displayed when an application is not responding. -->
    <string name="anr_process">Process <xliff:g id="process">%1$s</xliff:g> is not responding.\n\nWould you like to close it?</string>
    <!-- Button allowing the user to close an application that is not responding. This will kill the application. -->
    <string name="force_close">OK</string>

./frameworks/base/services/java/com/android/server/am/AppNotRespondingDialog.java:                    resid = com.android.internal.R.string.anr_activity_application;

### 超时定义

frameworks/base/services/java/com/android/server/am/ActivityManagerService.java

    // How long we wait until we timeout on key dispatching.
    static final int KEY_DISPATCHING_TIMEOUT = 5*1000;


### 超时检测

#### 1 在5秒内没有响应输入的事件（例如，按键按下，屏幕触摸） 

当前处于激活状态的应用程序会通过调用setInputWindows函数把把当前获得焦点的Activity窗口设置到InputDispatcher的mFocusedWindow中去，InputReader通过调用getEvent向Linux内核InputEvent驱动获取输入事件，经过对输入事件的检查以及结合屏幕方向进一步处理后，再通过调用notifyKey或notifyMotion方式通知InputDispatcher，InputDispatcher从mFocusedWindow获取当前激活状态的窗口，并通过调用sendDispatchSignal把输入事件发送给应用程序。InputDispatcher在发送输入事件前先检查当前激活的Activity窗口是否还在处理前一个输入事件，如果是的话，那就要等待它处理完前一个键盘事件后再来处理新的键盘事件，并且进行5秒超时判断，如果超时，就调用notifyANR通知WindowManager，WindowManager再通知ActivityManager进行处理。


**./base/services/input/InputDispatcher.cpp**

bool InputDispatcherThread::threadLoop() {

void InputDispatcher::dispatchOnce() {

void InputDispatcher::dispatchOnceInnerLocked(nsecs_t* nextWakeupTime) {

bool InputDispatcher::dispatchKeyLocked(nsecs_t currentTime, KeyEntry* entry,

findFocusedWindowTargetsLocked->

    // If there is no currently focused window and no focused application
    // If the currently focused window is paused then keep waiting.
    // If the currently focused window is still working on previous events then keep waiting.

int32_t InputDispatcher::handleTargetsNotReadyLocked(nsecs_t currentTime,

    if (currentTime >= mInputTargetWaitTimeoutTime) {
        onANRLocked(currentTime, applicationHandle, windowHandle,
                entry->eventTime, mInputTargetWaitStartTime);


onANRLocked->


    void InputDispatcher::doNotifyANRLockedInterruptible(
        CommandEntry* commandEntry) {
    mLock.unlock();

    nsecs_t newTimeout = mPolicy->notifyANR(
            commandEntry->inputApplicationHandle, commandEntry->inputWindowHandle);

    mLock.lock();

    resumeAfterTargetsNotReadyTimeoutLocked(newTimeout,
            commandEntry->inputWindowHandle != NULL
                    ? commandEntry->inputWindowHandle->getInputChannel() : NULL);
    }

**./base/services/java/com/android/server/wm/InputManager.java**

   /*
     * Callbacks from native.
     */
    private final class Callbacks {

        @SuppressWarnings("unused")
        public long notifyANR(InputApplicationHandle inputApplicationHandle,
                InputWindowHandle inputWindowHandle) {
            return mWindowManagerService.mInputMonitor.notifyANR(
                    inputApplicationHandle, inputWindowHandle);
        }

**./base/services/java/com/android/server/wm/InputMonitor.java**

    /* Notifies the window manager about an application that is not responding.
     * Returns a new timeout to continue waiting in nanoseconds, or 0 to abort dispatch.
     *
     * Called by the InputManager.
     */
    public long notifyANR(InputApplicationHandle inputApplicationHandle,

                    Slog.i(WindowManagerService.TAG, "Input event dispatching timed out sending to "
        if (appWindowToken != null && appWindowToken.appToken != null) {
            try {
                // Notify the activity manager about the timeout and let it decide whether
                // to abort dispatching or keep waiting.
                boolean abort = appWindowToken.appToken.keyDispatchingTimedOut();
                if (! abort) {
                    // The activity manager declined to abort dispatching.
                    // Wait a bit longer and timeout again later.
                    return appWindowToken.inputDispatchingTimeoutNanos;
                }
            } catch (RemoteException ex) {
            }
        }
        return 0; // abort dispatching


**./base/services/java/com/android/server/am/ActivityRecord.java**

    public boolean keyDispatchingTimedOut() {

        if (anrApp != null) {
            service.appNotResponding(anrApp, r, this,
                    "keyDispatchingTimedOut");
        }


**frameworks/base/services/java/com/android/server/am/ActivityManagerService.java**

    final void appNotResponding(ProcessRecord app, ActivityRecord activity,

        File tracesFile = dumpStackTraces(true, firstPids, processStats, lastPids);


#### 2 BroadcastReceiver在10秒内没有执行完毕

/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java


broadcastIntentLocked
->scheduleBroadcastsLocked

AmS: setBroadcastTimeoutLocked-> handleMessage:BROADCAST_TIMEOUT_MSG -> broadcastTimeoutLocked(true)->appNotResponding->dumpStackTraces


广播是在Android中一种广泛运用的用于应用程序之间传输信息的机制，Android中广播机制包括应用程序或框架发送广播消息，Android框架层ActivityManagerService服务分发管理广播信息，以及应用程序或框架注册广播接收器，并接收来它所监听的广播消息。ActivityManagerService服务分发管理广播信息包括无序以及有序两种模式。在无序模式下，ActivityManagerService对广播消息的分发是一种非同步的模式，即不等待一个广播接收器处理完广播消息，就会将广播消息分发给下一个广播接收器。而无序模式类似于一种同步模式，ActivityManagerService会等待一个广播接收器处理完这个广播消息，才会将广播消息分发给下一个广播接收器。有序和无序模式是针对动态注册的广播接收器来说的，而对于静态注册的广播接收器则一律是有序模式。

为了保证系统的响应性和稳定性， ActivityManagerService向各个应用程序或框架注册的广播接收器分发广播消息过程以及广播接收器处理广播消息的过程并不是无限期的，Android针对有序模式的广播接收器的处理设置了固定期限即10秒超时，保证每一个的Receiver的处理时长不超过规定时长，如果超时发生时，ActivityManagerService将会抛出应用程序无响应, 向用户报告当前的广播接收器在处理广播消息时存在无响应状况或者响应时间过长等状况。

Reference: http://blog.csdn.net/windskier/article/details/7251742

#### 3 Service在20秒内没有执行完毕

bumpServiceExecutingLocked -> serviceDoneExecutingLocked ->

handleMessage:SERVICE_TIMEOUT_MSG -> serviceTimeout -> appNotResponding

在Android中，Service是用于后台运行的执行任务的服务。调用者和服务之间没有联系，即使调用者退出了，服务依然在运行。服务默认都是在应用程序的主线程中运行的。如果Service要运行非常耗时或者可能被阻塞的操作时，将会严重影响应用程序的响应性。Android框架层ActivityManagerService服务负责服务的创建与管理。为了保证系统的响应性，Android为服务运行设置了固定期限即20秒超时，ActivityManagerService在收到用户启动服务请求时，将为用户程序创建并启动指定的服务，与此同时将调用系统发送消息服务指定在20秒后发出服务超时消息，应用服务应该在20秒内执行完成，并且通知ActivityManagerService服务执行完成并且删除此超时消息。如果此超时消息未能及时删除，ActivityManagerService将会抛出应用程序无响应错误。

![startService时的调用顺序](http://img.my.csdn.net/uploads/201109/20/0_1316483943xMyx.gif)

### 超时处理

appNotResponding中

#### 1 dumpStackTraces

File tracesFile = dumpStackTraces(true, firstPids, processStats, lastPids);
 
    final void appNotResponding(ProcessRecord app, ActivityRecord activity,

    /**
     * If a stack trace dump file is configured, dump process stack traces.
     * @param clearTraces causes the dump file to be erased prior to the new
     *    traces being written, if true; when false, the new traces will be
     *    appended to any existing file content.
     * @param firstPids of dalvik VM processes to dump stack traces for first
     * @param lastPids of dalvik VM processes to dump stack traces for last
     * @return file containing stack traces, or null if no dump file is configured
    public static File dumpStackTraces(boolean clearTraces, ArrayList<Integer> firstPids,

可以通过dalvik.vm.stack-trace-file 设置dumpStackTraces保存路径，默认为/data/anr/traces.txt

**保存内容：**

./base/services/java/com/android/server/am/ActivityManagerService.java

- First collect all of the stacks of the most important pids.

            if (firstPids != null) {
                try {
                    int num = firstPids.size();
                    for (int i = 0; i < num; i++) {
                        synchronized (observer) {
                            Process.sendSignal(firstPids.get(i), Process.SIGNAL_QUIT);
                            observer.wait(200);  // Wait for write-close, give up after 200msec
                        }
                    }
                } catch (InterruptedException e) {
                    Log.wtf(TAG, e);
                }
            }

实现方式为Process.sendSignal(firstPids.get(i), Process.SIGNAL_QUIT);


/dalvik/vm/SignalCatcher.cpp

signalCatcherThreadStart

static void* signalCatcherThreadStart(void* arg)

        switch (rcvd) {
        case SIGQUIT:
            handleSigQuit();
            break;
        case SIGUSR1:
            handleSigUsr1();
            break;
    #if defined(WITH_JIT) && defined(WITH_JIT_TUNING)
        case SIGUSR2:
            handleSigUsr2();
            break;
    #endif
    

保存位置：/data/anr/trace.txt->trace_<process-name>

**那么保存的进程有哪些呢？**


// Dump thread traces as quickly as we can, starting with "interesting" processes.

     firstPids.add(app.pid);

先保存本进程的PID

            for (int i = mLruProcesses.size() - 1; i >= 0; i--) {
                ProcessRecord r = mLruProcesses.get(i);
                if (r != null && r.thread != null) {
                    int pid = r.pid;
                    if (pid > 0 && pid != app.pid && pid != parentPid && pid != MY_PID) {
                        if (r.persistent) {
                            firstPids.add(pid);
                        } else {
                            lastPids.put(pid, Boolean.TRUE);
                        }
                    }
                }
            }

再保存运行的进程中，是永恒运行的进程。

所以保存的信息的进程包括：

    currentAPP
    system_server
    com.android.systemui
    com.android.phone
    sys.DeviceHealth

- Next measure CPU usage.

    Process.sendSignal(stats.pid, Process.SIGNAL_QUIT);

保存位置： logcat main buffer

    Slog.e(TAG, info.toString());


    E/ActivityManager(  344): ANR in com.android.development (com.android.development/.BadBehaviorActivity)
    E/ActivityManager(  344): Reason: keyDispatchingTimedOut
    E/ActivityManager(  344): Load: 10.17 / 10.07 / 10.06
    E/ActivityManager(  344): CPU usage from 22512ms to 29261ms later with 99% awake:
    E/ActivityManager(  344):   2.2% 344/system_server: 1.9% user + 0.2% kernel
    E/ActivityManager(  344):   1.3% 126/adbd: 0% user + 1.3% kernel
    E/ActivityManager(  344):   1.3% 19165/kworker/0:2: 0% user + 1.3% kernel
    E/ActivityManager(  344):   0% 9/sync_supers: 0% user + 0% kernel
    E/ActivityManager(  344):   0.1% 89/yaffs-bg-1: 0% user + 0.1% kernel
    E/ActivityManager(  344): 3.1% TOTAL: 0.5% user + 2.5% kernel
    E/ActivityManager(  344): CPU usage from 18421ms to 18974ms later:
    E/ActivityManager(  344):   8.9% 344/system_server: 8.9% user + 0% kernel
    E/ActivityManager(  344):     8.9% 433/InputDispatcher: 3.5% user + 5.3% kernel
    E/ActivityManager(  344):     3.5% 353/FinalizerDaemon: 3.5% user + 0% kernel
    E/ActivityManager(  344):     1.7% 359/er.ServerThread: 1.7% user + 0% kernel
    E/ActivityManager(  344):   1.3% 19165/kworker/0:2: 0% user + 1.3% kernel
    E/ActivityManager(  344): 10% TOTAL: 7.2% user + 3.6% kernel


#### Collect tombstones

           String tracesPath = SystemProperties.get("dalvik.vm.stack-trace-file", null);
            if (tracesPath != null && tracesPath.length() != 0) {
                File traceRenameFile = new File(tracesPath);
                String newTracesPath;
                int lpos = tracesPath.lastIndexOf (".");
                if (-1 != lpos)
                    newTracesPath = tracesPath.substring (0, lpos) + "_" + app.processName + tracesPath.substring (lpos);
                else
                    newTracesPath = tracesPath + "_" + app.processName;
                traceRenameFile.renameTo(new File(newTracesPath));

                Process.sendSignal(app.pid, 6);
                SystemClock.sleep(1000);
                Process.sendSignal(app.pid, 6);
                SystemClock.sleep(1000);
            }

位置： /data/tombstones/

tombstoneNoCrash_xx

## How to analyse

### 先看LOG CPU usage:

从LOG可以看出ANR的类型，CPU的使用情况，如果CPU使用量接近100%，说明当前设备很忙，有可能是CPU饥饿导致了ANR

如果CPU使用量很少，说明主线程被BLOCK了

如果IOwait很高，说明ANR有可能是主线程在进行I/O操作造成的

### check the trace log to see the status of each thread in the process and try to find where it hangs.

    /data/anr/traces.txt

    DALVIK THREADS:
    (mutexes: tll=0 tsl=0 tscl=0 ghl=0)
    "main" prio=5 tid=1 TIMED_WAIT
      | group="main" sCount=1 dsCount=0 obj=0x40a73538 self=0x1548e28
      | sysTid=8172 nice=0 sched=0/0 cgrp=ux handle=1074406760
      | schedstat=( 0 0 0 ) utm=33 stm=26 core=0
      at java.lang.VMThread.sleep(Native Method)
      at java.lang.Thread.sleep(Thread.java:1031)
      at java.lang.Thread.sleep(Thread.java:1013)
      at com.android.development.BadBehaviorActivity$6.onClick(BadBehaviorActivity.java:180)
      at android.view.View.performClick(View.java:3511)
      at android.view.View$PerformClick.run(View.java:14105)
      at android.os.Handler.handleCallback(Handler.java:605)
      **at android.os.Handler.dispatchMessage(Handler.java:92)**
      **at android.os.Looper.loop(Looper.java:137)**
      at android.app.ActivityThread.main(ActivityThread.java:4482)
      at java.lang.reflect.Method.invokeNative(Native Method)
      at java.lang.reflect.Method.invoke(Method.java:511)
      at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:787)
      at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:554)
      at dalvik.system.NativeStart.main(Native Method)

说明主线程在等待下条消息进入消息队列


## How to fix

Android应用程序通常是运行在一个单独的线程（例如，main）里。这意味着你的应用程序所做的事情如果在主线程里占用了太长的时间的话，就会引发ANR对话框，因为你的应用程序并没有给自己机会来处理输入事件或者Intent广播。

因此，运行在主线程里的任何方法都尽可能少做事情。特别是，Activity应该在它的关键生命周期方法（如onCreate()和onResume()）里尽可能少的去做创建操作。潜在的耗时操作，例如网络或数据库操作，或者高耗时的计算如改变位图尺寸，应该在子线程里（或者以数据库操作为例，通过异步请求的方式）来完成。然而，不是说你的主线程阻塞在那里等待子线程的完成——也不是调用Thread.wait()或是Thread.sleep()。替代的方法是，主线程应该为子线程提供一个Handler，以便完成时能够提交给主线程。以这种方式设计你的应用程序，将能保证你的主线程保持对输入的响应性并能避免由于5秒输入事件的超时引发的ANR对话框。这种做法应该在其它显示UI的线程里效仿，因为它们都受相同的超时影响。

IntentReceiver执行时间的特殊限制意味着它应该做：在后台里做小的、琐碎的工作如保存设定或者注册一个Notification。和在主线程里调用的其它方法一样，应用程序应该避免在BroadcastReceiver里做耗时的操作或计算。但不再是在子线程里做这些任务（因为BroadcastReceiver的生命周期短），替代的是，如果响应Intent广播需要执行一个耗时的动作的话，应用程序应该启动一个Service。顺便提及一句，你也应该避免在Intent Receiver里启动一个Activity，因为它会创建一个新的画面，并从当前用户正在运行的程序上抢夺焦点。如果你的应用程序在响应Intent广播时需要向用户展示什么，你应该使用Notification Manager来实现。

一般来说，在应用程序里，100到200ms是用户能感知阻滞的时间阈值。因此，这里有一些额外的技巧来避免ANR，并有助于让你的应用程序看起来有响应性。

- 如果你的应用程序为响应用户输入正在后台工作的话，可以显示工作的进度（ProgressBar和ProgressDialog对这种情况来说很有用）。 
特别是游戏，在子线程里做移动的计算。
- 如果你的应用程序有一个耗时的初始化过程的话，考虑可以显示一个Splash Screen或者快速显示主画面并异步来填充这些信息。在这两种情况下，你都应该显示正在进行的进度，以免用户认为应用程序被冻结了。

## Reference

* [Designing for Responsiveness](http://developer.android.com/guide/practices/responsiveness.html)
* [Android ANR问题的分析和解决](http://wenku.it168.com/d_000083535.shtml)
* [android anr分析方法](http://blog.csdn.net/gemmem/article/details/7446421)
* [android Application Component研究之BroadcastReceiver](http://blog.csdn.net/windskier/article/details/7251742)
* [Android Intent原理分析](http://apps.hi.baidu.com/share/detail/38586591)
* [Service启动过程过程详解](http://blog.csdn.net/cloudwu007/article/details/6792589)