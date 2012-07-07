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

- BroadcastReceiver在10秒内没有执行完毕
( If **BroadcastReceiver** does not finish executing within 10 sec, ActivityManager triggers ANR.)

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

#### WindowManager



./base/services/java/com/android/server/wm/InputMonitor.java:                    Slog.i(WindowManagerService.TAG, "Input event dispatching timed out sending to "

    /* Notifies the window manager about an application that is not responding.
     * Returns a new timeout to continue waiting in nanoseconds, or 0 to abort dispatch.
     *
     * Called by the InputManager.
     */
    public long notifyANR(InputApplicationHandle inputApplicationHandle,





./base/services/java/com/android/server/am/ActivityRecord.java:    public boolean keyDispatchingTimedOut() {
./base/services/java/com/android/server/am/ActivityRecord.java:                    info.putString("shortMsg", "keyDispatchingTimedOut");
./base/services/java/com/android/server/am/ActivityRecord.java:                    "keyDispatchingTimedOut");



    /** Returns the key dispatching timeout for this application token. */
    public long getKeyDispatchingTimeout() {
        synchronized(service) {
            ActivityRecord r = getWaitingHistoryRecordLocked();
            if (r != null && r.app != null
                    && (r.app.instrumentationClass != null || r.app.usingWrapper)) {
                return ActivityManagerService.INSTRUMENTATION_KEY_DISPATCHING_TIMEOUT;
            }

            return ActivityManagerService.KEY_DISPATCHING_TIMEOUT;
        }
    }

### 超时处理

## How to analyse

check the trace log to see the status of each thread in the process and try to find where it hangs.

    /data/anr/traces.txt

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