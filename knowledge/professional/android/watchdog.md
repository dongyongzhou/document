---
layout: master
title: Android Watchdog
---


## Overview

Android Watchdog位于android framework层中，属于一种软件Watchdog实现。

### Watchdog主要作用： 

- 接收系统内部reboot请求,重启系统
- 监护SystemServer进程,防止系统死锁

### 内容

- Watchdog内部架构
- Watchdog启动流程
- Watchdog工作流程

## Watchdog内部架构

    frameworks/base/services/java/com/android/server/Watchdog.java

### Watchdog内部主要class：

HeartbeatHandler、RebootReceiver、RebootRequestReceiver

**HeartbeatHandler**：此为WatchDog的核心，负责对各个监护对象进行监护。Used for scheduling monitor callbacks and checking memory usage.

**RebootReceiver**：负责接收由**AlarManagerService**发出的PendingIntent,并进行系统重启。该PendingIntent为WatchDog内部创建，"com.android.service.Watchdog.REBOOT"。

**RebootRequestReceiver**：负责接收系统内部发出的重启Intent消息，并进行系统重启。

### Watchdog内部主要接口函数

checkReboot、rebootSystem、Monitor、addMonitor

**checkReboot**：判断是否需要重启系统。由HeartbeatHandler、RebootReceiver、RebootRequestReceiver调用。

**rebootSystem**：调用PowerManager的reboot接口重启系统。由checkReboot调用

**Monitor**：每个被监护对象必须要**实现的接口**，由WatchDog在运行中调用，以实现监护功能

**addMonitor**：将实现了monitor接口的监护对象注册到WatchDog服务中。


## Watchdog启动流程


WatchDog是在**SystemServer进程**中被初始化和启动的。在SystemServer被Start时，各种Android服务被注册和启动，其中也包括了WatchDog的初始化和启动。

Frameworks/base/services/java/com/android/server/SystemServer.java

    @Override
    public void run() {
            ....
            Slog.i(TAG, "Init Watchdog");
            Watchdog.getInstance().init(context, battery, power, alarm,
                    ActivityManagerService.self());

            ....
                Watchdog.getInstance().start();



Watchdog本身继承Thread，是一个线程类。此为WatchDog初始化。

.在SystemServer Run函数的后半段，将检查系统是否已经准备好运行第三方代码，并通过SystemReady接口通知系统已经就绪。在ActivityManagerService的SystemReady接口的CallBack函数中实现WatchDog的启动Watchdog.getInstance().start();


## Watchdog监护

### WatchDog监护对象

**实现**

- 实现WatchDog.Monitor接口，这个接口中只有一个monitor函数
- 将该对象注册到WatchDog服务中,在初始化中作如下处理：

    Watchdog.getInstance().addMonitor(this); 

在Android中WatchDog运行在SystemServer进程,对其进行监护

**而其中监护的服务为以下三个**

- ActivityManagerService
- WindowManagerService
- PowerMangerService

**ActivityManagerService**

    public final class ActivityManagerService extends ActivityManagerNative
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {


    private ActivityManagerService() {
        ...
        // Add ourself to the Watchdog monitors.
        Watchdog.getInstance().addMonitor(this);
        ...

    /** In this method we try to acquire our lock to make sure that we have not deadlocked */
    public void monitor() {
        synchronized (this) { }
    }


该接口函数其实内部并不做任何处理，只是去锁一下对象，然后返回。如果对象没有死锁，则过程会很顺利，若对象死锁，则该函数就会挂在这里。
其它两个Service对象实现的monitor接口函数与Activity类似，也同样是去获取一下锁而已


###WatchDog监护流程

**Step 1 :** WatchDog启动之后，开始运行run函数

- While循环周期，周期性发命令：mHandler.sendEmptyMessage(MONITOR);周期为TIME_TO_WAIT的默认时间为30s。此为第一次等待时间，WatchDog判断对象是否死锁的最长处理时间为1Min。

    public void run() {
        boolean waitedHalf = false;
        while (true) {
            mCompleted = false;
            mHandler.sendEmptyMessage(MONITOR);
            synchronized (this) {
                long timeout = TIME_TO_WAIT;

                // NOTE: We use uptimeMillis() here because we do not want to increment the time we
                // wait while asleep. If the device is asleep then the thing that we are waiting
                // to timeout on is asleep as well and won't have a chance to run, causing a false
                // positive on when to kill things.
                long start = SystemClock.uptimeMillis();
                while (timeout > 0 && !mForceKillSystem) {
                    try {
                        wait(timeout);  // notifyAll() is called when mForceKillSystem is set
                    } catch (InterruptedException e) {
                        Log.wtf(TAG, e);
                    }
                    timeout = TIME_TO_WAIT - (SystemClock.uptimeMillis() - start);
                }

                if (mCompleted && !mForceKillSystem) {
                    // The monitors have returned.
                    waitedHalf = false;
                    continue;
                }


            }

**Step 2 :** HeartbeatHandler负责接收并处理MONITOR的Message

- WatchDog同时会等待30秒，等待HeartbeatHandler的处理结果。然后才会进行下一步动作。
- HeartbeatHandler依次去调用监护对象的monitor接口，实现对其的监护。
- 如果监护的对象都正常，则会很快运行下去，并对mCompleted赋值为true，表示对象正常返回。mCompleted值初始为false。

    final class HeartbeatHandler extends Handler {
         
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MONITOR: {
                    // See if we should force a reboot.
                    int rebootInterval = mReqRebootInterval >= 0
                            ? mReqRebootInterval : Settings.Secure.getInt(
                            mResolver, Settings.Secure.REBOOT_INTERVAL,
                            REBOOT_DEFAULT_INTERVAL);
                    if (mRebootInterval != rebootInterval) {
                        mRebootInterval = rebootInterval;
                        // We have been running long enough that a reboot can
                        // be considered...
                        checkReboot(false);
                    }

                    final int size = mMonitors.size();
                    for (int i = 0 ; i < size ; i++) {
                        mCurrentMonitor = mMonitors.get(i);
                        mCurrentMonitor.monitor();
                    }

                    synchronized (Watchdog.this) {
                        mCompleted = true;
                        mCurrentMonitor = null;
                    }
                } break;
            }
        }
    }

**Step 3 :** WatchDog run函数

- 如果所有对象在30s内能够返回，则会得到mCompleted = true; 则本次监护就结束，返回继续下一轮监护。
- 如果在30s内，monitor对象未能返回，mCompleted 值即为false，则会运行到以下语句。会调用ActivityManagerService.java中的dumpStackTraces接口函数。


                if (!waitedHalf) {
                    // We've waited half the deadlock-detection interval.  Pull a stack
                    // trace and wait another half.
                    ArrayList<Integer> pids = new ArrayList<Integer>();
                    pids.add(Process.myPid());
                    ActivityManagerService.dumpStackTraces(true, pids, null, null);
                    waitedHalf = true;
                    continue;
                }

- 在dumpStackTraces接口中，主要会对SystemServer进程的stackTrace的信息dump出来，以及检测目前运行App的CPU使用率。由SystemServer进程发送一个SIGNAL_QUIT的进程信号：

                            synchronized (observer) {
                                Process.sendSignal(stats.pid, Process.SIGNAL_QUIT);
                                observer.wait(200);  // Wait for write-close, give up after 200msec
                            }

- 该动作发生在第一次等待的30s时间内，monitor对象未返回，由于在调用完ActivityManagerService.java的dumpStackTraces接口函数后，将waitedHalf赋值为true。并返回继续下一轮监护。
- 若紧接着的下一轮监护，在30s内monitor对象依旧未及时返回，此时mCompleted=false， waitedHalf = true; 相应的语句部分都不会运行，则会直接运行到下面部分。这表示系统的监护对象有死锁现象发生，SystemServer进程需要kill并重启。即60s, WatchDog判断对象是否死锁的最长处理时间为1Min。

         // If we got here, that means that the system is most likely hung.
            // First collect stack traces from all threads of the system process.
            // Then kill this process so that the system will restart.

            // Pass !waitedHalf so that just in case we somehow wind up here without having
            // dumped the halfway stacks, we properly re-initialize the trace file.
            final File stack = ActivityManagerService.dumpStackTraces(
                    !waitedHalf, pids, null, null);


            // Give some extra time to make sure the stack traces get written.
            // The system's been hanging for a minute, another second or two won't hurt much.
            SystemClock.sleep(2000);

            // Pull our own kernel thread stacks as well if we're configured for that
            if (RECORD_KERNEL_THREADS) {
                dumpKernelStackTraces();
            }


             .....
            // Only kill the process if the debugger is not attached.
            if(!Debug.isDebuggerConnected()) {
               if(SystemProperties.getInt("sys.watchdog.disabled", 0) == 0) {
                  Process.sendSignal(Process.myPid(), 6);
                  SystemClock.sleep(2000);
                  Process.sendSignal(Process.myPid(), 6);
                  SystemClock.sleep(2000);
              ....
                  Slog.w(TAG, "*** WATCHDOG KILLING SYSTEM PROCESS: " + name);
                  Process.killProcess(Process.myPid());
                  System.exit(10);

- 在剩下的30s内，做一些收尾工作，如重新初始化trace file。最后直接将SystemServer进程kill，并且退出系统。Init进程会重新启动SystemServer进程，让其回到可用状态。


## The End

