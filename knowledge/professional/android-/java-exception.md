---
layout: master
title: Java Exception
---


## Overview

Java exception

##3 Java Exception

REFERENCE

* [深入探索 高效的Java异常处理框架](http://rq2-79.iteye.com/blog/1408331)

###3.1 Overview

**异常**是指在程序中出现的异常状况.

**异常机制**是指当程序出现错误后，程序如何处理。具体来说，异常机制提供了程序退出的安全通道。当出现错误后，程序执行的流程发生改变，程序的控制权转移到异常处理器。

###3.2 传统的处理异常的办法

传统的处理异常的办法是，函数返回一个特殊的结果来表示出现异常（通常这个特殊结果是大家约定俗称的），调用该函数的程序负责检查并分析函数返回的结果。

这样做有如下的弊端：例如函数返回-1代表出现异常，但是如果函数确实要返回-1这个正确的值时就会出现混淆；可读性降低，将程序代码与处理异常的代码混爹在一起；由调用函数的程序来分析错误，这就要求客户程序员对库函数有很深的了解。

###3.3 Java 异常

在Java中异常被抽象成一个叫做Throwable的类。

- 遇到错误，方法立即结束，并不返回一个值；同时，抛出一个异常对象
- 调用该方法的程序也不会继续执行下去，而是搜索一个可以处理该异常的异常处理器，并执行其中的代码

**异常的继承结构**：

- 基类为Throwable
- Error和Exception继承Throwable
- RuntimeException和IOException等继承Exception
- 具体的RuntimeException继承RuntimeException

![From Android Cookbook](http://androidcookbook.com/seam/resource/graphicImage/Throwable.png)
![From ANDORID学习指南](http://android.yaohuiji.com/wp-content/uploads/2010/12/image26.png)


**Error与Exception**
   
- Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。
- Exception是程序本身可以处理的异常，这种异常分两大类运行时异常和非运行时异常。程序中应当尽可能去处理这些异常。

**运行时异常和非运行时异常**

- 运行时异常都是RuntimeException类及其子类异常，如NullPointerException、 IndexOutOfBoundsException等，这些异常是**不检查异常**，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引 起的，程序应该从逻辑角度尽可能避免这类异常的发生。
- 非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

**未检查异常（unchecked）和已检查异常（checked）**

Error和RuntimeException及其子类成为未检查异常（unchecked）

其它异常成为已检查异常（checked）,这类异常则可以在程序编写和编译阶段加以事先检查和处理。

RuntimeException我们不好在程序编写阶段加以事先处理，

相应的我们把除检验异常(可检查异常)之外的异常称之为非检验异常，包括Error和RuntimeException ，非检验异常可以捕捉也可以不捕捉，更多的时候我们不捕捉，因为捕捉了我们也没办法处理，譬如程序运行时发生了一个VirtualMachineError异常，虚拟机都出错了，作为运行在虚拟机内的程序又有什么办法处理呢？

程序只需要捕捉和处理检验异常。

**3.3.1 Error体系**

Error类体系描述了Java运行系统中的内部错误以及资源耗尽的情形。

其中如果程序出错并不是由程序本身引起的，而是硬件等其他原因引起的，我们称之为Error，一般情况下Error一旦产生，对程序来说都是致命的错误，程序本身无能为力，所以我们可以不对Error作出任何处理和响应。

应用程序不应该抛出这种类型的对象（一般是由虚拟机抛出）。如果出现这种错误，除了尽力使程序安全退出外，在其他方面是无能为力的。所以，在进行程序设计时，应该更关注Exception体系。

**3.3.2  Exception体系**

异常如果是由程序引起的我们称之为Exception，Exception又分两种，我们把运行时才会出现的异常叫做 RuntimeException，RuntimeException我们不好在程序编写阶段加以事先处理，而其他异常则可以在程序编写和编译阶段加以事先检查和处理，我们把这种异常叫做检验异常。

Exception体系包括RuntimeException体系和其他非RuntimeException的体系

- 3.3.2.1 RuntimeException

RuntimeException体系包括**错误的类型转换、数组越界访问和试图访问空指针**等等。
处理RuntimeException的原则是：如果出现RuntimeException，那么一定是程序员的错误。例如，可以通过检查数组下标和数组边界来避免数组越界访问异常。

- 3.3.2.2 其他（IOException等等）

这类异常一般是外部错误，例如试图从文件尾后读取数据等，这并不是程序本身的错误，而是在应用环境中出现的外部错误。

###3.4 Java 异常的使用方法

**3.4.1 声明方法抛出异常**

**WHY**

方法是否抛出异常与方法返回值的类型一样重要。假设方法抛出异常确没有声明该方法将抛出异常，那么客户程序员可以调用这个方法而且不用编写处理异常的代码。那么，一旦出现异常，那么这个异常就没有合适的异常控制器来解决。

RuntimeException与Error可以在任何代码中产生，它们不需要由程序员显示的抛出，一旦出现错误，那么相应的异常会被自动抛出。而已检查异常是由程序员抛出的，这分为两种情况：

为什么抛出的异常一定是已检查异常？

- 客户程序员调用会抛出异常的库函数（库函数的异常由库程序员抛出）；
- 客户程序员自己使用throw语句抛出异常。遇到Error，程序员一般是无能为力的；遇到RuntimeException，那么一定是程序存在逻辑错误，要对程序进行修改（相当于调试的一种方法）；只有已检查异常才是程序员所关心的，程序应该且仅应该抛出或处理已检查异常。

注意：覆盖父类某方法的子类方法不能抛出比父类方法更多的异常，所以，有时设计父类的方法时会声明抛出异常，但实际的实现方法的代码却并不抛出异常，这样做的目的就是为了方便子类方法覆盖父类方法时可以抛出异常。

**3.4.2 如何抛出异常**

语法：throw（略）

**抛出什么异常？**

对于一个异常对象，真正有用的信息时**异常的对象类型**，而异常对象本身毫无意义。比如一个异常对象的类型是ClassCastException，那么这个类名就是唯一有用的信息。所以，在选择抛出什么异常时，最关键的就是**选择异常的类名**能够明确说明异常情况的类。

异常对象通常有两种构造函数：

- 一种是无参数的构造函数；
- 另一种是带一个字符串的构造函数，这个字符串将作为这个异常对象除了类型名以外的额外说明。

**创建自己的异常**：

当Java内置的异常都不能明确的说明异常情况的时候，需要创建自己的异常。需要注意的是，唯一有用的就是类型名这个信息，所以不要在异常类的设计上花费精力。

###3.5 捕获异常

如果一个异常没有被处理，那么，对于一个非图形界面的程序而言，该程序会被中止并输出异常信息；对于一个图形界面程序，也会输出异常的信息，但是程序并不中止，而是返回用Ы缑娲?硌?分小?BR> 

**3.5.1 语法：**

try、catch和finally

![](http://android.yaohuiji.com/wp-content/uploads/2010/12/image_thumb28.png)


**try语句块**，表示要尝试运行代码，try语句块中代码受异常监控，其中代码发生异常时，会抛出异常对象。

**catch语句块**会捕获try代码块中发生的异常并在其代码块中做异常处理，catch语句带一个Throwable类型的参数，表示可捕获异常类型。当 try中出现异常时，catch会捕获到发生的异常，并和自己的异常类型匹配，若匹配，则执行catch块中代码，并将catch块参数指向所抛的异常对 象。catch语句可以有多个，用来匹配多个中的一个异常，一旦匹配上后，就不再尝试匹配别的catch块了。通过异常对象可以获取异常发生时完整的 JVM堆栈信息，以及异常信息和异常发生的原因等。

**finally语句块**是紧跟catch语句后的语句块，这个语句块总是会在方法返回前执行，而不管是否try语句块是否发生异常。并且这个语句块总是在方法返回前执行。目的是给程序一个补救的机会。这样做也体现了Java语言的健壮性。

控制器模块必须紧接在try块后面。若掷出一个异常，异常控制机制会搜寻参数与异常类型相符的第一个控制器随后它会进入那个catch
从句，并认为异常已得到控制。一旦catch 从句结束对控制器的搜索也会停止。

**异常处理的几条规则**：

- try用于定义可能发生异常的代码段，这个代码块被称为监视区域，所有可能出现检验异常的代码写在这里。
- catch代码段紧跟在try代码段后面，中间不能有任何其他代码。
- try后面可以没catch代码段，这实际上是放弃了捕捉异常，把异常捕捉的任务交给调用栈的上一层代码。
- try后面可以有一个或者多个catch代码段，如果有多个catch代码段那么程序只会进入其中某一个catch。
- catch捕捉的多个异常之间有继承关系的话，要先捕捉子类后捕捉父类。
- finally代码段可以要也可以不要。
- 如果try代码段没有产生异常，那么finally代码段会被立即执行，如果产生了异常，那么finally代码段会在catch代码段执行完成后立即执行。
- 可以只有try和finally没有catch。

**3.5.1 异常处理做什么？**

对于Java来说，由于有了垃圾收集，所以异常处理并不需要回收内存。但是依然有一些资源需要程序员来收集，比如文件、网络连接和图片等资源。

**应该声明方法抛出异常还是在方法中捕获异常**？

原则：捕捉并处理哪些知道如何处理的异常，而传递哪些不知道如何处理的异常

**再次抛出异常**

为什么要再次抛出异常？

在本级中，只能处理一部分内容，有些处理需要在更高一级的环境中完成，所以应该再次抛出异常。这样可以使每级的异常处理器处理它能够处理的异常。

异常处理流程

对应与同一try块的catch块将被忽略，抛出的异常将进入更高的一级。

一般情况下，异常对象唯一有用的信息就是类型信息。但使用异常带字符串的构造函数时，这个字符串还可以作为额外的信息。调用异常对象的getMessage()、toString()或者printStackTrace()方法可以分别得到异常对象的额外信息、类名和调用堆栈的信息。并且后一种包含的信息是前一种的超集。

Throwable类中的常用方法

    getCause()：返回抛出异常的原因。如果 cause 不存在或未知，则返回 null。
    getMessage()：返回异常的消息信息。
    printStackTrace()：对象的堆栈跟踪输出至错误输出流，作为字段 System.err 的值。

**3.5.2 throw、throws关键字**

throw关键字是用于方法体内部，用来抛出一个Throwable类型的异常。如果抛出了检查异常，则还应该在方法头部声明方法可能抛出的异常类型。该 方法的调用者也必须检查处理抛出的异常。如果所有方法都层层上抛获取的异常，最终JVM会进行处理，处理也很简单，就是打印异常消息和堆栈信息。如果抛出 的是Error或RuntimeException，则该方法的调用者可选择处理该异常。有关异常的转译会在下面说明。

throws关键字用于方法体外部的方法声明部分，用来声明方法可能会抛出某些异常。仅当抛出了检查异常，该方法的调用者才必须处理或者重新抛出该异常。 当方法的调用者无力处理该异常的时候，应该继续抛出，而不是囫囵吞枣一般在catch块中打印一下堆栈信息做个勉强处理。下面给出一个简单例子，看看如何 使用这两个关键字：

**3.5.2 常见异常**

- ArrayIndexOfBoundsException	数组下标越界异常
- ClassCastException	强制转换类失败异常
- IllegalArgumentException	方法参数类型传入异常
- IllegalStateException	非法的设备状态异常
- NullPointException	传说中的空指针异常，如果一个对象不存在，你有对这个对象使用点操作，那么就会出现该异常
- NumberFormatException	把字符串转成数字失败时出现的数字格式异常
- AssertionError	断言错误
- ExceptionInInitializerError	试图初始化静态变量或者静态初始化块时抛出
- StackOverflowError	栈溢出错误
- NoClassDefFoundError	找不到指定的类错误


###3.6 to Android

在发生语言级错误时，应该使用Java Exception机制管理Error。

如果所有方法都层层上抛获取的异常，最终AndroidRuntime（DALVI）会进行处理，处理也很简单，就是打印异常消息和堆栈信息。如果抛出 的是Error或RuntimeException，则该方法的调用者可选择处理该异常。有关异常的转译会在下面说明。

因为Android上的应用和服务都是Java的代码，它的Error和 Exception都是沿用Java的，比如Error有 AssertionError，VirtualMachineError，OutOfMemoryError和其他的Error类。Exception有 RuntimeException和IOException，

**Question**

- Exception如何传递?
- Android最终如何处理未处理的Exception有没有保存trace或者tombstone?

ANS: /data/anr/下有trace信息。
/data/tombstones/下没trace信息。

- Android最终如何处理未处理的Exception，如何输出提示框的？

android.app.Application和java.lang.Thread.UncaughtExceptionHandler。

Application：用来管理应用程序的全局状态。在应用程序启动时Application会首先创建，然后才会根据情况(Intent)来启动相应的Activity和Service。可在自定义加强版的Application中注册未捕获异常处理器。

Thread.UncaughtExceptionHandler：线程未捕获异常处理器，用来处理未捕获异常。如果程序出现了未捕获异常，默认会弹出系统中强制关闭对话框。我们需要实现此接口，并注册为程序中默认未捕获异常处理。这样当未捕获异常发生时，就可以做一些个性化的异常处理操作。

**错误提示定义**:

    ./frameworks/base/core/res/res/values/strings.xml:    <string name="aerr_application">Unfortunately, <xliff:g id="application">%1$s</xliff:g> has stopped.</string>
    ./frameworks/base/core/res/res/values/strings.xml:    <string name="aerr_process">Unfortunately, the process <xliff:g id="process">%1$s</xliff:g> has
    ./frameworks/base/core/res/res/values-xxxx/strings.xml:    <string name="aerr_application" msgid="932628488013092776">"Unfortunately, <xliff:g id="APPLICATION">%1$s</xliff:g> has stopped."</string>

    <!-- Text of the alert that is displayed when an application has crashed. -->
    <string name="aerr_application">Unfortunately, <xliff:g id="application">%1$s</xliff:g> has stopped.</string>
    <!-- Text of the alert that is displayed when an application has crashed. -->
    <string name="aerr_process">Unfortunately, the process <xliff:g id="process">%1$s</xliff:g> has
        stopped.</string>

**错误提示显示**:

    ./frameworks/base/services/java/com/android/server/am/AppErrorDialog.java:                    com.android.internal.R.string.aerr_application,

**错误提示调用**:

    ./frameworks/base/services/java/com/android/server/am/ActivityManagerService.java:            Dialog d = new AppErrorDialog(mContext, res, proc);
    ./frameworks/base/services/java/com/android/server/am/ActivityManagerService.java:            if (res == AppErrorDialog.FORCE_QUIT_AND_REPORT) 

调用机制：

handleApplicationCrash->crashApplicationv->mHandler.sendMessage(SHOW_ERROR_MSG)

     * Used by {@link com.android.internal.os.RuntimeInit} to report when an application crashes.

处理对话框：

    ActivityManagerService. final Handler mHandler{

        public void handleMessage(Message msg) {
            switch (msg.what) {
            case SHOW_ERROR_MSG: {
                HashMap data = (HashMap) msg.obj;
                synchronized (ActivityManagerService.this) {
                    ProcessRecord proc = (ProcessRecord)data.get("app");
                    if (proc != null && proc.crashDialog != null) {
                        Slog.e(TAG, "App already has crash dialog: " + proc);
                        return;
                    }
                    AppErrorResult res = (AppErrorResult) data.get("result");
                    if (!mSleeping && !mShuttingDown) {
                        Dialog d = new AppErrorDialog(mContext, res, proc);
                        d.show();
                        proc.crashDialog = d;
                    } else {
                        // The device is asleep, so just pretend that the user
                        // saw a crash dialog and hit "force quit".
                        res.set(0);
                    }
                }

                ensureBootCompleted();
            } break;

更上一层

    ./base/core/java/com/android/internal/os/RuntimeInit.java

RuntimeInit.UncaughtHandler->handleApplicationCrash

    private static class UncaughtHandler implements Thread.UncaughtExceptionHandler {

                // Bring up crash dialog, wait for it to be dismissed
                ActivityManagerNative.getDefault().handleApplicationCrash(
                        mApplicationObject, new ApplicationErrorReport.CrashInfo(e));
            } catch (Throwable t2) {
                try {
                    Slog.e(TAG, "Error reporting crash", t2);
                } catch (Throwable t3) {
                    // Even Slog.e() fails!  Oh well.
                }
            } finally {
                // Try everything to make sure this process goes away.
                Process.killProcess(Process.myPid());
                System.exit(10);
            }


可见RuntimeInit也是继承UncaughtExceptionHandler接收处理未捕捉异常。通过实现 UncaughtExceptionHandler就可以达到目的。

Use this to log a message when a thread exits due to an uncaught exception.  
The framework catches these for the main threads, 
so this should only matter for threads created by applications.


#### Collect logs..

Mainlogs

/dalvik/vm/Thread.cpp

static void threadExitUncaughtException(Thread* self, Object* group)

/dalvik/vm/Exception.cpp

->void dvmLogExceptionStackTrace()

W/dalvikvm( 2686): threadid=1: thread exiting with uncaught exception (group=0x40a5c9d8)
E/AndroidRuntime( 2686): FATAL EXCEPTION: main
E/AndroidRuntime( 2686): com.android.development.BadBehaviorActivity$BadBehaviorException: Whatcha gonna do, whatcha gonna do

eventlogs:


    public void handleApplicationCrash(IBinder app, ApplicationErrorReport.CrashInfo crashInfo) {
        ProcessRecord r = findAppProcess(app, "Crash");
        final String processName = app == null ? "system_server"
                : (r == null ? "unknown" : r.processName);

        EventLog.writeEvent(EventLogTags.AM_CRASH, Binder.getCallingPid(),
                processName,
                r == null ? -1 : r.info.flags,
                crashInfo.exceptionClassName,
                crashInfo.exceptionMessage,
                crashInfo.throwFileName,
                crashInfo.throwLineNumber);

        addErrorToDropBox("crash", r, processName, null, null, null, null, null, crashInfo);

        crashApplication(r, crashInfo);
    }