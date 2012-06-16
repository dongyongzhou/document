---
layout: master
title: Android Startup
---

## Overview

Android 启动流程可划分为四大部分

- Linux内核启动
- Android init启动
- Zygote启动
- System Server启动

##2 Linux内核启动

###2.1 上电

###2.2 运行Boot ROM 

- 找到到第一阶段bootloader位置
- 加载第一阶段bootloader到internal RAM
- 跳转到第一阶段bootloader内存位置，并启动

###2.3 运行第一阶段bootloader

- **初始化外部RAM**
- 找到第二阶段bootloader
- 加载第二阶段bootloader到外部RAM
- 跳转到第二阶段bootloader内存位置，并启动

###2.4 运行第二阶段bootloader

- 设置文件系统Setup file systems (一般是Flash 设备)
- 有**选择性地驱动**显示、网络、辅助存储器以及其他设备
- 使能其他CPU功能
- 使能 low-level 内存保护
- 选择性地加载安全保护措施 (e.g. ROM validation code)
- 找到Linux内核
- 加载Linux内核到内存
- 将Linux内核对启动参数放置到内存，使能内核知道启动时应该运行什么
- 跳转到Linux内核内存地址，并启动LInux

###2.5 运行Linux 内核

- 在内存中生成一个表，描述物理内存的布局
- 初始化并驱动输入设备
- 初始化并驱动磁盘控制器（特指MTD），并映射可用的块设备到内存
- 初始化高级电源管理支持（APM）
- 初始化中断处理程序: Interrupt Descriptor Table (IDT), Global Descriptor Table (GDT), and Programmable
Interrupt Controllers (PIC)
- 重置浮点运算单元(FPU)
- 从实模式切换到保护模式(i.e. enable memory protection)
- 初始化段寄存器segmentation registers和 临时的堆栈
- 零初始化内存
- 解压内核镜像
- 初始化临时的内核页表并使能分页
- 为0进程设置内核模式栈
- 用空的中断处理程序填充IDT
- 用系统参数初始化第一个页框
- 确定CPU模型
- 用GDT和IDT地址初始化相应的寄存器
- 初始化并启动内核

   * 调度程序Scheduler
   * 内存分区Memory zones
   * 伙伴系统分配器Buddy system allocator
   * 中断描述符表IDT
   * 软中断SoftIRQs
   * 日期和时间Date and Time
   * Slab分配器Slab allocator
   * . . . .

- 创建进程1（/init）并运行


## Android init启动

### Source code

- init的源文件多放在system/core/init下
- init.rc放在system/core/rootdir/init.rc下
- 不同平台（device）的init.rc放在device/下

### Overview

- 所有其他进程的祖先都是/init(PID=1)
- Custom initialization script (with its own language)， Different than more traditional /etc/inittab and SysV-init-levels initialization options
- The init program never exists (if it were, the kernel would panic), so it continues to monitor started services

### init分析

When started by the kernel, init parses and executes commands from two files:

- /init.rc - provides generic initialization instructions (see system/core/rootdir/init.rc)
- /init.board-name.rc - provides machine-specific initialization instructions - sometimes overriding /init.rc

    /init.goldfish.rc for the emulator
    /init.trout.rc for HTC’s ADP1
    /init.herring.rc for Samsung’s Nexus S (see device/samsung/crespo/init.herring.rc)

### init’s language

The init’s language consists of four broad classes of statements (see system/core/init/readme.txt)

#### **Actions** - named sequences of commands queued to be executed on a unique trigger

    `on <trigger>
     <command>
     <command>
     <command>`

#### **Triggers** - named events that trigger actions

* `early-init, init, early-fs, fs, post-fs, early-boot, boot` - built-in stages of init
* `<name>=<value>` - fires when a system property is set to `<name>=<value>`
* `device-added-<path>` - fires when a device node at `<path>` is added
* `device-removed-<path>` - fires when a device node at `<path>` is removed
* `service-exited-<name>` - fires when a service by `<name>` exists

#### **Commands** - a command to be queued and run

* chdir `<directory>` - change working directory
* chmod `<octal-mode> <path>` - change file access permissions
* chown `<owner> <group> <path>` - change file user and group ownership
* chroot `<directory>` - change process root directory
* class_start `<serviceclass>` - start all non-running services of the specified class
* class_stop `<serviceclass>` - stop all running services of the specified class
* domainname `<name>` - set the domain name.
* exec `<path> [ <argument> ]*` - fork and execute `<path> <argument> ... (blocks init until
exec returns)`
* export `<name> <value>` - set globally visible environment variable `<name> =<value>`
* `hostname <name>` - set the host name
* `ifup <interface>` - bring the network interface <interface> online
* `import <filename>` - parse and process <filename> init configuration file (extends the current script)
* `insmod <path> `- install the kernel module at `<path>`
* `loglevel <level>` - initialize the logger to `<level>`
* `mkdir <path> [mode] [owner] [group]` - create a directory at `<path> `and optionally change its default permissions `(755) and user (root) / group (root) `ownership
* `mount <type> <device> <dir> [ <mountoption> ]*` - attempt to mount named <device> at
`<dir> `with the optional `<mountoption>`s
* `setprop <name> <value> - set system property <name>=<value>`
* `setrlimit <resource> <cur> <max> - set the rlimit for a <resource>`
* `start <service> - start named <service> if it is not already running`
* `stop <service> - stop name <service> if it is currently running`
* `symlink <target> <path> - symbolically link <path> to <target>`
* `sysclktz <mins_west_of_gmt> - set the system clock base (0 for GMT)`
* `trigger <event> - trigger an event - i.e. call one action from another`
* `write <path> <string> [ <string> ]* - write arbitrary <string>`s to file by specified
`<path>`

#### Services - persistent daemon programs to be launched (optionally) restarted if they exit

    `service <name> <pathname> [ <argument> ]*
    <option>
    <option>
    ...`

#### Options - modifiers to services, modifying how/when they are run re/launched

* critical - a device-critical service - devices goes into recovery if the service exits more than 4 times in 4 minutes
* disabled - not started by default - i.e. it has to be started explicitly by name
* `setenv <name> <value> - set the environment variable <name>=<value> in the service`
* `socket <name> <dgram|stream|seqpacket> <perm> [ <user> [ <group> ] ] - create a unix
domain socket named /dev/socket/<name> and pass its fd to the service`
* `user <username> - change service’s effective user ID to <username>`
* `group <groupname> [ <groupname> ]* - change service’s effective group ID to <groupname> (additional groups are supplemented via setgroups())`
* oneshot - ignore service exist (default is to auto-restart)
* `class <name> - set the class name of the service (defaults to default) so that all services of a particular class can be started/stopped together`
* onrestart - execute a command on restart


### init启动分析


### The default init.rc script

1. Starts ueventd
2. Initializes the system clock and logger
3. Sets up global environment
4. Sets up the file system (mount points and symbolic links)
5. Configures kernel timeouts and scheduler
6. Configures process groups
7. Mounts the file systems
8. Creates a basic directory structure on /data and applies permissions
9. Applies permissions on /cache
10. Applies permissions on certain /proc points
11. Initializes local network (i.e. localhost)
12. Configures the parameters for the low memory killer
13. Applies permissions for systemserver and daemons
14. Defines TCP buffer sizes for various networks
15. Configures and (optionally) loads various daemons (i.e. services): ueventd, console, adbd, **servicemanager**, vold, netd, debuggerd, rild, **zygote (which in turn starts system_server)**, mediaserver, bootanimation(one time), and various Bluetooth daemons (like dbus-daemon, bluetoothd, etc.), installd, racoon, mtpd, keystore


### Nexus S’ init.herring.rc additionally

1. Sets up product info
2. Initializes device-driver-specific info, file system structures, and permissions: battery, wifi, phone, uart_switch,
GPS, radio, bluetooth, NFC, lights
3. Initializes and (re)mounts file systems
4. Loads additional device-specific daemons

### ServiceManager启动


    service servicemanager /system/bin/servicemanager
    class core
    user system
    group system
    critical
    onrestart restart zygote
    onrestart restart media
    onrestart restart surfaceflinger
    onrestart restart drm

servicemanager代码在

	system\core\libsysutils\src


    ServiceManager的启动为什么要在Service启动之前呢？

    因为在C/S模式中，他会作为S（服务端）要运行着，为的是让之后启动的Service调用他的addService（）函数来注册。ServiceManager跟Service之间的通信是基于Binder的IPC机制

frameworks/base/core/java/android/os/ServiceManager.java

里面的几个函数：

- addservice():用来给service注册用的，
- checkservice():一般主要给应用程序来寻找所需要用到的service的Binder的，
- getservice():用来提供service的IBinder实例的。

## Zygote启动

###1. Zygote starts from /init.rc

    service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
        class main
        socket zygote stream 666
        onrestart write /sys/android_power/request_state wake
        onrestart write /sys/power/state on
        onrestart restart media
        onrestart restart netd

###2. This translates to frameworks/base/cmds/app_process/app_main.cpp:main()
###3. The command app_process then launches frameworks/base/core/java/com/android/internal/os/ZygoteInit.java in a Dalvik VM via frameworks/base/core/jni/AndroidRuntime.cpp:start()
即是使用Dalvik启动system server

    int main(int argc, const char* const argv[])

        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;

    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit",
                startSystemServer ? "start-system-server" : "");


![DVM实例启动的全过程](http://hi.csdn.net/attachment/201203/17/0_1331996781qLjc.gif)

###4. ZygoteInit.main() then

1. Registers for zygote socket
2. Pre-loads classes defined in frameworks/base/preloaded-classes (1800+)
3. Pre-loads resources preloaded_drawables and preloaded_color_state_lists from frameworks/base/core/
4. Runs garbage collector (to clean the memory as much as possible)
5. Forks itself to **start systemserver**，先是启动Dalvik Virtual Machine
6. Starts listening for requests to fork itself for other apps


## System Server启动

###1. When Zygote forks itself to launch the systemserver process

    frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:startSystemServer()
    public static void main(String argv[]) {
            if (argv[1].equals("start-system-server")) {
                startSystemServer();
            } 

    private static boolean startSystemServer()

        /* Hardcoded command line to start the system server */
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,3001,3002,3003,3006,3007,3008",
            "--capabilities=130104352,130104352",
            "--runtime-init",
            "--nice-name=system_server",
            "com.android.server.SystemServer",


            /* Request to fork the system server process */
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.debugFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);

it executes 

    frameworks/base/services/java/com/android/server/SystemServer.java:main()

###2. The SystemServer:java.main() method loads android_servers JNI lib from *frameworks/base/services/jni* and invokes *init1()* native method

    public static void main(String[] args) {

        System.loadLibrary("android_servers");
        init1(args);


    /**
     * This method is called from Zygote to initialize the system. This will cause the native
     * services (SurfaceFlinger, AudioFlinger, etc..) to be started. After that it will call back
     * up into init2() to start the Android services.
     */

    native public static void init1(String[] args);

###3. Before init1() runs, the JNI loader first runs 

    frameworks/base/services/jni/onload.cpp:JNI_OnLoad(),

 which registers native services - to be used as JNI counterparts to Java-based service manager loaded later

###4. Now frameworks/base/services/jni/com_android_server_SystemServer.cpp:init1() is
invoked, which simply wraps a call to frameworks/base/cmds/system_server/library/system_init.cpp:system_5. The system_init.cpp:system_init() function

frameworks/base/services/jni/com_android_server_SystemServer.cpp:init1()

    static JNINativeMethod gMethods[] = {
    /* name, signature, funcPtr */
    { "init1", "([Ljava/lang/String;)V", (void*) android_server_SystemServer_init1 },
    };

    static void android_server_SystemServer_init1(JNIEnv* env, jobject clazz)
    {
        system_init();
    }

frameworks/base/cmds/system_server/library/system_init.cpp:system_5

a. First starts native services (some optionally):

- i. frameworks/base/services/surfaceflinger/SurfaceFlinger.cpp
- ii. frameworks/base/services/sensorservice/SensorService.cpp
- iii. frameworks/base/services/audioflinger/AudioFlinger.cpp
- iv. frameworks/base/media/libmediaplayerservice/MediaPlayerService.cpp
- v. frameworks/base/camera/libcameraservice/CameraService.cpp
- vi. frameworks/base/services/audioflinger/AudioPolicyService.cpp

b. Then goes back to frameworks/base/services/java/com/android/server/SystemServer.java:init2(),
again via frameworks/base/core/jni/AndroidRuntime.cpp:start() JNI call

    jclass clazz = env->FindClass("com/android/server/SystemServer");
    if (clazz == NULL) {
        return UNKNOWN_ERROR;
    }

    jmethodID methodId = env->GetStaticMethodID(clazz, "init2", "()V");
    if (methodId == NULL) {
        return UNKNOWN_ERROR;
    }
    env->CallStaticVoidMethod(clazz, methodId);


###6. The SystemServer.java:init2() method then starts Java service managers in **a separate thread (ServerThread)**, readies them, and registers each one with 

frameworks/base/services/java/com/android/server/SystemServer.java

    public static final void init2() {
        Slog.i(TAG, "Entered the Android system server!");
        Thread thr = new ServerThread();
        thr.setName("android.server.ServerThread");
        thr.start();
    }

    class ServerThread extends Thread {
    private static final String TAG = "SystemServer";
    private static final String ENCRYPTING_STATE = "trigger_restart_min_framework";
    private static final String ENCRYPTED_STATE = "1";

    ContentResolver mContentResolver;

    void reportWtf(String msg, Throwable e) {
        Slog.w(TAG, "***********************************************");
        Log.wtf(TAG, "BOOT FAILURE " + msg, e);
    }

    @Override
    public void run() {
            ....
            Slog.i(TAG, "Power Manager");
            power = new PowerManagerService();
            ServiceManager.addService(Context.POWER_SERVICE, power);
           ....
     
    frameworks/base/core/java/android/os/ServiceManager:addService()

(which in turn delegates to to ServiceManagerNative.java, which effectively talks to servicemanager daemon previously started by init)

- a. frameworks/base/services/java/com/android/server/PowerManagerService.java
- b. frameworks/base/services/java/com/android/server/am/ActivityManagerService.java
- c. frameworks/base/services/java/com/android/server/TelephonyRegistry.java
- d. frameworks/base/services/java/com/android/server/PackageManagerService.java
- e. frameworks/base/services/java/com/android/server/BatteryService.java
- f. frameworks/base/services/java/com/android/server/VibratorService.java
- g. etc

###7. Finally frameworks/base/services/java/com/android/server/am/ActivityManagerService.java:finishBooting() sets sys.boot_completed=1 and sends out

- a. a broadcast intent with **android.intent.action.PRE_BOOT_COMPLETED** action (to give apps a
chance to reach to boot upgrades)
- b. an activity intent with **android.intent.category.HOME** category to launch the Home (or Launcher)
application
- c. a broadcast intent with **android.intent.action.BOOT_COMPLETED** action, which launches applications subscribed to this intent (while using android.permission.RECEIVE_BOOT_COMPLETED)

## Reference

*[android 启动流程](http://hi.baidu.com/billycoder/blog/item/76023c1f103c5b9486d6b65c.html)

