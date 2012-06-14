---
layout: master
title: Android trace file
---

##1 Overview

Android Dalvik应用收到该信号后，会打印改应用中所有线程的状态以及调用栈情况，并且并不是强制退出。这些状态通常保存在一个特定的叫做trace的文件中。

Traces.txt holds java level stacktraces

The stack trace of the application is save in /data/anr/traces.txt 

研究内容：

- Trace file如何生成？
- Trace file内容及解释？

##2 Trace file如何生成

Trace文件是 android davik 虚拟机在收到异常终止信号 （SIGQUIT）时产生的。 

###2.1 触发条件

最经常的触发条件是 android应用中产生了 ANR或FC （force close）。

实质

/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java

setBroadcastTimeoutLocked->
handleMessage:BROADCAST_TIMEOUT_MSG ->
broadcastTimeoutLocked->appNotResponding->dumpStackTraces

serviceTimeout->appNotResponding->dumpStackTraces

ActivityStack

handleMessage: PAUSE_TIMEOUT_MSG/LAUNCH_TICK_MSG
logAppTooSlow->dumpStackTraces

./frameworks/base/services/java/com/android/server/am/DeviceMonitor.java

    private static final File BASE = new File("/data/anr/");


dumpStackTraces

// First collect all of the stacks of the most important pids.

// Next measure CPU usage.

// sending signal 

                        synchronized (observer) {
                            Process.sendSignal(firstPids.get(i), Process.SIGNAL_QUIT);
                            observer.wait(200);  // Wait for write-close, give up after 200msec
                        }


/frameworks/base/core/java/android/os/Process.java:    public static final int SIGNAL_QUIT = 3;


###2.2 服务对象

由于是该文件的产生是在 DVM里，所以只有运行 dvm实例的进程（如普通的java应用，java服务等）才会产生该文件

android 本地应用 （native app，指运行在 android lib层，用c/c++编写的linux应用、库、服务等）在收到 SIGQUIT时是不会产生 trace文件的。

可以在终端通过adb发送SIGQUIT给应用来生成trace文件。

###2.3 实现过程

**相关实现代码文件**

Android ICS 实现文件后缀是 .cpp, 早期实现是.c

    dalvik/vm/init.cpp
    davik/vm/SignalCatcher.cpp
    dalvik/vm/Thread.cpp

**实现过程**


**Step #1:**  DVM初始化时，设置信号屏蔽字，屏蔽要特殊处理的信号(SIGQUIT, SIGUSR1, SIGUSR2)。由于信号处理方式是进程范围起作用的， 这意味着该进程里所有的线程都将屏蔽该信号。 实现代码在init.c中如下：

    dalvik/vm/init.cpp
    std::string dvmStartup(int argc, const char* const argv[],
        bool ignoreUnrecognized, JNIEnv* pEnv)
    {
     ....
    /* configure signal handling */
    if (!gDvm.reduceSignals)
        blockSignals();
     ...  
    } 

	/*
     * Configure signals.  We need to block SIGQUIT so that the signal only
     * reaches the dump-stack-trace thread.
     *
     * This can be disabled with the "-Xrs" flag.
     */
    static void blockSignals()
    {
    sigset_t mask;
    int cc;

    sigemptyset(&mask);
    sigaddset(&mask, SIGQUIT);
    sigaddset(&mask, SIGUSR1);      // used to initiate heap dump
    #if defined(WITH_JIT) && defined(WITH_JIT_TUNING)
    sigaddset(&mask, SIGUSR2);      // used to investigate JIT internals
    #endif
    //sigaddset(&mask, SIGPIPE);
    cc = sigprocmask(SIG_BLOCK, &mask, NULL);
    assert(cc == 0);

    if (false) {
        /* TODO: save the old sigaction in a global */
        struct sigaction sa;
        memset(&sa, 0, sizeof(sa));
        sa.sa_sigaction = busCatcher;
        sa.sa_flags = SA_SIGINFO;
        cc = sigaction(SIGBUS, &sa, NULL);
        assert(cc == 0);
    }
    }

**Step #2：** DVM 生成单独的信号**处理线程**，用来对三个信号做特殊处理 （init.cpp）：

    dalvik/vm/init.cpp

    /*
     * Do non-zygote-mode initialization.  This is done during VM init for
     * standard startup, or after a "zygote fork" when creating a new process.
     */
    bool dvmInitAfterZygote()
    {
    ....
    /* start signal catcher thread that dumps stacks on SIGQUIT */
    if (!gDvm.reduceSignals && !gDvm.noQuitHandler) {
        if (!dvmSignalCatcherStartup())
            return false;
    }
    ....

    davik/vm/SignalCatcher.cpp
    /*
     * Crank up the signal catcher thread.
     *
     * Returns immediately.
     */
    bool dvmSignalCatcherStartup()
    {
    gDvm.haltSignalCatcher = false;

    if (!dvmCreateInternalThread(&gDvm.signalCatcherHandle,
                "Signal Catcher", signalCatcherThreadStart, NULL))
        return false;

    return true;
    }

DVM调用dvmCreateInternalThread()来生成一个新的内部线程来专门处理dvm进程里的信号。 

dvmCreateInternalThread()其实是使用pthread_create()来产生新的线程。 

该线程的处理函数是 signalCatcherThreadStart()。  

（dvm里所谓的内部线程，就是用来帮助dvm实现本身使用的线程，比如信号处理线程，binder线程，Compiler线程，JDWP线程等，而不是应用程序申请的线程）

    /*  
     * Sleep in sigwait() until a signal arrives.  
     */  
    static void* signalCatcherThreadStart(void* arg)  
    {  
    ...  
    /* set up mask with signals we want to handle */  
    sigemptyset(&mask);  
    sigaddset(&mask, SIGQUIT);  
    sigaddset(&mask, SIGUSR1);  
    #if defined(WITH_JIT) && defined(WITH_JIT_TUNING)  
    sigaddset(&mask, SIGUSR2);  
    #endif  
    ...  
    while (true) {  
    ...  
        /*
         * Signals for sigwait() must be blocked but not ignored.  We
         * block signals like SIGQUIT for all threads, so the condition
         * is met.  When the signal hits, we wake up, without any signal
         * handlers being invoked.
         *
         * When running under GDB we occasionally return from sigwait()
         * with EINTR (e.g. when other threads exit).
         */

    loop:  
        cc = sigwait(&mask, &rcvd);  
        ...  
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
        ...  
    }  

- 首先设置我们要处理的信号集(SIGQUIT, SIGUSR1, SIGUSR2)
- 然后 调用 sigwait()。 我们知道sigwait()会在当前的线程里重新打开指定的信号屏蔽字屏蔽的信号集。  
dvm在启动时，首先在整个进程里设置信号屏蔽字屏蔽掉三个信号，sigwait()的调用，使的这三个信号只在 SignalCatcher线程里响应。

dvm对三个信号分别所做的特殊用途：

1. SIGUSR1 被用来做手工垃圾收集。处理函数是 HandleSigUsr1()

    /*
     * Respond to a SIGUSR1 by forcing a GC.
     */
    static void handleSigUsr1()
    {
        LOGI("SIGUSR1 forcing GC (no HPROF)");
        dvmCollectGarbage();
    }

2. SIGUSR2 被用来做 JIT的调试。如果JIT下编译时打开，收到SIGUSR2时dvm会dump出相关的调试信息。
 
   
     #if defined(WITH_JIT) && defined(WITH_JIT_TUNING)
    /* Sample callback function for dvmJitScanAllClassPointers */
    void printAllClass(void *ptr)
    {
        ClassObject **classPP = (ClassObject **) ptr;
        LOGE("class %s", (*classPP)->descriptor);
    }
    /*
     * Respond to a SIGUSR2 by dumping some JIT stats and possibly resetting
     * the code cache.
     */
    static void handleSigUsr2()
    {
        static int codeCacheResetCount = 0;
        gDvmJit.receivedSIGUSR2 ^= true;
        if ((--codeCacheResetCount & 7) == 0) {
            /* Dump all class pointers in the traces */
            dvmJitScanAllClassPointers(printAllClass);
            gDvmJit.codeCacheFull = true;
        } else {
            dvmCompilerDumpStats();
            /* Stress-test unchain all */
            dvmJitUnchainAll();
            LOGD("Send %d more signals to reset the code cache",
                 codeCacheResetCount & 7);
        }
        dvmCheckInterpStateConsistency();
    }
     #endif

SIGUSR1、SIGUSR2两个信号都仅用于DVM的内部实现的调试。可以在终端通过adb发送SIGUSR1和SIGUSR2信号来观察它的行为。

3.  SIGQUIT 用来输出trace文件，以记录异常终止是dvm的上下文信息.

/*
 * Respond to a SIGQUIT by dumping the thread stacks.  Optionally dump
 * a few other things while we're at it.
 *
 * Thread stacks can either go to the log or to a file designated for holding
 * ANR traces. ** If we're writing to a file**, we want to do it in one shot,
 * so we can use a single O_APPEND write instead of contending for exclusive
 * access with flock().  There may be an advantage in resuming the VM
 * before doing the file write, so we don't stall the VM if disk I/O is
 * bottlenecked.
 *
 * If JIT tuning is compiled in, dump compiler stats as well.
 */

    static void handleSigQuit()
    {
    char* traceBuf = NULL;
    size_t traceLen;

    dvmSuspendAllThreads(SUSPEND_FOR_STACK_DUMP);

    dvmDumpLoaderStats("sig");

    if (gDvm.stackTraceFile == NULL) {
        /* just dump to log */
        DebugOutputTarget target;
        dvmCreateLogOutputTarget(&target, ANDROID_LOG_INFO, LOG_TAG);
        dvmDumpAllThreadsEx(&target, true);
    } else {
        /* write to memory buffer */
        FILE* memfp = open_memstream(&traceBuf, &traceLen);
        if (memfp == NULL) {
            LOGE("Unable to create memstream for stack traces");
            traceBuf = NULL;        /* make sure it didn't touch this */
            /* continue on */
        } else {
            logThreadStacks(memfp);
            fclose(memfp);
        }
    }

    #if defined(WITH_JIT) && defined(WITH_JIT_TUNING)
    dvmCompilerDumpStats();
    #endif

    if (false) dvmDumpTrackedAllocations(true);

    dvmResumeAllThreads(SUSPEND_FOR_STACK_DUMP);

    if (traceBuf != NULL) {
        /*
         * We don't know how long it will take to do the disk I/O, so put us
         * into VMWAIT for the duration.
         */
        ThreadStatus oldStatus = dvmChangeStatus(dvmThreadSelf(), THREAD_VMWAIT);

        /*
         * Open the stack trace output file, creating it if necessary.  It
         * needs to be world-writable so other processes can write to it.
         */
        int fd = open(gDvm.stackTraceFile, O_WRONLY | O_APPEND | O_CREAT, 0666);
        if (fd < 0) {
            LOGE("Unable to open stack trace file '%s': %s",
                gDvm.stackTraceFile, strerror(errno));
        } else {
            ssize_t actual = write(fd, traceBuf, traceLen);
            if (actual != (ssize_t) traceLen) {
                LOGE("Failed to write stack traces to %s (%d of %zd): %s",
                    gDvm.stackTraceFile, (int) actual, traceLen,
                    strerror(errno));
            } else {
                LOGI("Wrote stack traces to '%s'", gDvm.stackTraceFile);
            }
            close(fd);
        }

        free(traceBuf);
        dvmChangeStatus(dvmThreadSelf(), oldStatus);
    }
    }

- 首先查看有没有有指定 trace输出文件，没有就将trace信息打印到log里。如果有，就先将trace信息打印到内存文件traceBuf中，然后再将改内存文件traceBuf内容输出到指定trace文件中。 

**为什么指定了trace文件后，不直接打印trace信息到trace文件中呢。** 

原因是trace文件实际上记录的是**当前运行的所有的线程的上下文信息**。他需要暂停所有的线程才能输出。 dvmSuspendAllThreads(SUSPEND_FOR_STACK_DUMP);的调用正式这个目的。这个操作代价是很高的，它把当前所有的线程都停了下来。执行的时间越短，对正常运行的线程的影响越小。 输出信息到内存比直接到外部文件要快得多。所以 dvm采取了先输出到内存，马上恢复线程程dvmResumeAllThreads(SUSPEND_FOR_STACK_DUMP);，然后就可以慢慢的输出到外部文件里了。

而这真正的输出信息实现在 logThreadStacks()中：

    /*
     * Dump the stack traces for all threads to the supplied file, putting
     * a timestamp header on it.
     */
    static void logThreadStacks(FILE* fp)
    {
        DebugOutputTarget target;
    
        dvmCreateFileOutputTarget(&target, fp);
    
        pid_t pid = getpid();
        time_t now = time(NULL);
        struct tm* ptm;
    #ifdef HAVE_LOCALTIME_R
        struct tm tmbuf;
        ptm = localtime_r(&now, &tmbuf);
    #else
        ptm = localtime(&now);
    #endif
        dvmPrintDebugMessage(&target,
            "\n\n----- pid %d at %04d-%02d-%02d %02d:%02d:%02d -----\n",
            pid, ptm->tm_year + 1900, ptm->tm_mon+1, ptm->tm_mday,
            ptm->tm_hour, ptm->tm_min, ptm->tm_sec);
        printProcessName(&target);
        dvmPrintDebugMessage(&target, "\n");
        dvmDumpAllThreadsEx(&target, true);
        fprintf(fp, "----- end %d -----\n", pid);
    }

该函数打印了trace文件的框架，其输出类似如下所示：

    ----- pid 26013 at 2012-06-14 09:59:28 -----
    Cmd line: com.android.developmen

显示当前dvm进程的进程id，名字，输出的时间。最重要的所有线程的上下文信息是有函数 dvmDumpAllThreadsEx()里实现的，该函数定义在 thread.c里

    dalvik/vm/Thread.cpp

    /*
     * Print information about all known threads.  Assumes they have been
     * suspended (or are in a non-interpreting state, e.g. WAIT or NATIVE).
     *
     * If "grabLock" is true, we grab the thread lock list.  This is important
     * to do unless the caller already holds the lock.
     */
    void dvmDumpAllThreadsEx(const DebugOutputTarget* target, bool grabLock)
    {
        Thread* thread;
    
        dvmPrintDebugMessage(target, "DALVIK THREADS:\n");
    
    #ifdef HAVE_ANDROID_OS
        dvmPrintDebugMessage(target,
            "(mutexes: tll=%x tsl=%x tscl=%x ghl=%x)\n",
            gDvm.threadListLock.value,
            gDvm._threadSuspendLock.value,
            gDvm.threadSuspendCountLock.value,
            gDvm.gcHeapLock.value);
    #endif

        if (grabLock)
            dvmLockThreadList(dvmThreadSelf());

        thread = gDvm.threadList;
        while (thread != NULL) {
            dvmDumpThreadEx(target, thread, false);

            /* verify link */
            assert(thread->next == NULL || thread->next->prev == thread);

            thread = thread->next;
        }

        if (grabLock)
            dvmUnlockThreadList();
    }


## Trace file内容及解释？


    ----- pid 26013 at 2012-06-14 09:59:28 -----
    Cmd line: com.android.developmen

显示当前dvm进程的进程id，名字，输出的时间。

    DALVIK THREADS:
    (mutexes: tll=0 tsl=0 tscl=0 ghl=0)
    "main" prio=5 tid=1 TIMED_WAIT
      | group="main" sCount=1 dsCount=0 obj=0x40a7fc40 self=0x1ab2e38
      | sysTid=26013 nice=0 sched=0/0 cgrp=default handle=1074406760
      | schedstat=( 0 0 0 ) utm=30 stm=11 core=0
      at java.lang.VMThread.sleep(Native Method)
      at java.lang.Thread.sleep(Thread.java:1031)
      at java.lang.Thread.sleep(Thread.java:1013)
      at com.android.development.BadBehaviorActivity.onCreate(BadBehaviorActivity.java:118)
      at android.app.Activity.performCreate(Activity.java:4465)
      at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1049)
      at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:1928)
      at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:1989)
      at android.app.ActivityThread.access$600(ActivityThread.java:126)
      at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1155)
      at android.os.Handler.dispatchMessage(Handler.java:99)
      at android.os.Looper.loop(Looper.java:137)
      at android.app.ActivityThread.main(ActivityThread.java:4482)
      at java.lang.reflect.Method.invokeNative(Native Method)
      at java.lang.reflect.Method.invoke(Method.java:511)
      at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:787)
      at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:554)
      at dalvik.system.NativeStart.main(Native Method

trace文件中thread信息

1. 第一行是**固定的头**, 指明下面的都是当前运行的 dvm thread ：“DALVIK THREADS:”
2. 第二行输出的是该**进程里各种线程互斥量的值**。
3. 第三行输出分别是**线程的名字**（“main”），线程优先级（“prio=5”），线程id（“tid=1”） 以及线程的类型（“TIMED_WAIT”）【others such as NATIVE】
4. 第四行分别是**线程所述的线程组** （“main”），线程被正常挂起的次处（“sCount=1”），线程因调试而挂起次数（”dsCount=0“），当前线程所关联的java线程对象（”obj=0x40a7fc40“）以及该线程本身的地址（“self=0x1ab2e38”）。
5. 第五行显示**线程调度信息**。 分别是该线程在linux系统下得本地线程id （“ sysTid=26013”），线程的调度有优先级（“nice=0”），调度策略（sched=0/0），优先组属（“cgrp=default”）以及处理函数地址（“handle=1074406760”）
6 第六行 显示更多**该线程当前上下文**，分别是调度状态（从 /proc/[pid]/task/[tid]/schedstat读出）（最新可使用"/proc/self/task/%d/schedstat"）（“schedstat=( 0 0 0 )”），以及该线程运行信息 ，它们是线程用户态下使用的时间值(单位是jiffies）（“utm=30”）， 内核态下得调度时间值（“stm=11”），以及最后运行改线程的cup标识（“core=0”）；
7.后面几行输出该线程**调用栈**。


