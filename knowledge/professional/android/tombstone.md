---
layout: master
title: Android Tombstone
---

## Overview

VM generates a **crash report tombston**e when there is a processor-defined exception (signal from the kernel – **SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT**).

Once the core processor sees the problem, the kernel sends a signal to the debuggerd daemon socket in user space (system/core/debuggerd.c and bionic/linker/debugger.c).

The daemon generates a Tombstone Log (/data/tombstones/tombstone_xx) by getting a stack dump from each user process and the current CPU register value.

这里将研究Android Tombstone机制以及相应的异常机制

- exception如何产生
- SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT如何传递？
- Tombstone如何产生？
- Tombstone如何解析？

## exception如何产生？

To be continued...

### Tombstone 原因-Framework Reboot

- Watchdog killing system process
- Fatal exception in one of system_server’s threads
- Excessive JNI references


## SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT如何传递？

To be continued...

## Tombstone如何产生？
 
### Two Files.

    bionic/linker/debugger.c
    system/core/debuggerd.c 


### bionic/linker/debugger.c

**Step 1**： android Dynamic Linker 

Android 的加载/链接器linker 主要用于实现共享库的加载与链接。它支持应用程序对库函数的隐式和显式调用。

- 对于隐式调用，应用程序的编译与静态库大致相同，只是在静态链接的时候通过--dynamic-linker /system/bin/linker 指定动态链接器，（该信息将被存放在ELF文件的.interp节中，内核执行目标映像文件前将通过该信息加载并运行相应的解释器程序linker.）并链接相应的共享库。与ld.so不同的是，Linker目前没有提供Lazy Binding机制，所有外部过程引用都在映像执行之前解析。
- 对于显式调用，可以同过linker中提供的接口dlopen，dlsym,dlerror和dlclose来动态加载和链接共享库。

代码bionic/linker/linker.c

生成/system/bin/linker

也就是说，在加载过程时已为进程设定了信号处理方式。

**Step 2**： Called when a signal is received from Kernel, uses socket() to connect to The “android:debuggerd” socket, and write()s to the socket

__linker_init()  (bionic/linker/linker.c) -> debugger_init() (bionic/linker/debugger.c)-> debugger_signal_handler()  

    void debugger_init()
    {
        struct sigaction act;
        memset(&act, 0, sizeof(act));
        act.sa_sigaction = debugger_signal_handler;
        act.sa_flags = SA_RESTART | SA_SIGINFO;
        sigemptyset(&act.sa_mask);

        sigaction(SIGILL, &act, NULL);
        sigaction(SIGABRT, &act, NULL);
        sigaction(SIGBUS, &act, NULL);
        sigaction(SIGFPE, &act, NULL);
        sigaction(SIGSEGV, &act, NULL);
        sigaction(SIGSTKFLT, &act, NULL);
        sigaction(SIGPIPE, &act, NULL);
    }

采用Socke方式进行进程间通信

/*
 * Catches fatal signals so we can ask debuggerd to ptrace us before
 * we crash.
 */

    void debugger_signal_handler(int n, siginfo_t* info, void* unused)
    {
    unsigned tid;
    int s;

    logSignalSummary(n, info);

    tid = gettid();
    s = socket_abstract_client("android:debuggerd", SOCK_STREAM);

    if(s >= 0) {
        /* debugger knows our pid from the credentials on the
         * local socket but we need to tell it our tid.  It
         * is paranoid and will verify that we are giving a tid
         * that's actually in our process
         */
        int  ret;

        RETRY_ON_EINTR(ret, write(s, &tid, sizeof(unsigned)));
        if (ret == sizeof(unsigned)) {
            /* if the write failed, there is no point to read on
             * the file descriptor. */
            RETRY_ON_EINTR(ret, read(s, &tid, 1));
            notify_gdb_of_libraries();
        }
        close(s);
    }

    /* remove our net so we fault for real when we return */
    signal(n, SIG_DFL);
}

可以看到logSignalSummary将输出如下示例信息：

    F/libc    ( 1373): Fatal signal 11 (SIGSEGV) at 0x0000055d (code=0)

### system/core/debuggerd.c 

**Step 1**:The debuggerd daemon creates a socket server android:debuggerd and loops forever,waiting for some client to write into the socket  

    main()(system/core/debuggerd/debuggerd.c)

**Step 2**:Dump stack trace and registers in /data/tombstone/ 

    handle_crashing_process()->engrave_tombstone() (system/core/debuggerd/debuggerd.c)

        if(WIFSTOPPED(status)){
            n = WSTOPSIG(status);
            switch(n) {
            case SIGSTOP:
                XLOG("stopped -- continuing\n");
                n = ptrace(PTRACE_CONT, tid, 0, 0);
                if(n) {
                    LOG("ptrace failed: %s\n", strerror(errno));
                    goto done;
                }
                continue;

            case SIGABRT:
                isAnr = true;
            case SIGILL:
            case SIGBUS:
            case SIGFPE:
            case SIGSEGV:
            case SIGSTKFLT: {
                XLOG("stopped -- fatal signal\n");
                need_cleanup = engrave_tombstone(cr.pid, tid, debug_uid, n, isAnr);
                kill(tid, SIGSTOP);
                goto done;
            }

            default:
                XLOG("stopped -- unexpected signal\n");
                goto done;
            }
        } else {
            XLOG("unexpected waitpid response\n");
            goto done;
        }

### 内容

#### 保存位置

    static int find_and_open_tombstone(bool isAnr)
    snprintf(path, sizeof(path), TOMBSTONE_DIR"/tombstone%s_%02d", isAnr == true? "NoCrash":"", oldest);

- 如果是ANR类型引起的，即SIGABRT(源自anrNoResponding)，那么就会生成

    case SIGABRT:
         isAnr = true;

/tombstoneNoCrash_xx

- 如果不是ANR类型引起的，即SIGILL、SIGBUS、SIGFPE、SIGSEGV、SIGSTKFLT。那么就会生成
        
/tombstone_xx

#### 保存内容

    dump_crash_report(fd, pid, tid, true);
    dump_crash_report(fd, pid, tid, true);
    dump_logs(fd, pid, true);
    dump_sibling_thread_report(fd, pid, tid);
    dump_logs(fd, pid, false);


    void dump_crash_banner(int tfd, unsigned pid, unsigned tid, int sig)
    {
    char data[1024];
    char *x = 0;
    FILE *fp;

    sprintf(data, "/proc/%d/cmdline", pid);
    fp = fopen(data, "r");
    if(fp) {
        x = fgets(data, 1024, fp);
        fclose(fp);
    }

    _LOG(tfd, false,
         "*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***\n");
    dump_build_info(tfd);
    _LOG(tfd, false, "pid: %d, tid: %d  >>> %s <<<\n",
         pid, tid, x ? x : "UNKNOWN");

    if(sig) dump_fault_addr(tfd, tid, sig);
    }

    

    void dump_crash_report(int tfd, unsigned pid, unsigned tid, bool at_fault)
    dump_registers(tfd, tid, at_fault);
    parse_elf_info(milist, tid);
    dump_pc_and_lr(tfd, tid, milist, stack_depth, at_fault);
    dump_stack_and_code(tfd, tid, milist, stack_depth, sp_list, at_fault);
    dump_dalvik(tfd, milist, tid, at_fault);



    static void dump_logs(int tfd, unsigned pid, bool tailOnly)
    {
        dump_log_file(tfd, pid, "/dev/log/system", tailOnly);
        dump_log_file(tfd, pid, "/dev/log/main", tailOnly);
    }

#### tombstones 例子

 *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
    Build fingerprint: 'xxx'
    pid: 1373, tid: 1373  >>> com.android.development <<<
    signal 11 (SIGSEGV), code 0 (?), fault addr 0000055d
     r0 00000000  r1 0000000b  r2 0000055d  r3 0000000b
     r4 4bbe49d8  r5 00958e38  r6 00000000  r7 00000025
     r8 be8955e0  r9 4baded9c  10 00000008  fp be8955f4
     ip 4021ae3c  sp be8955c0  lr 401db0df  pc 400bd8d0  cpsr 20000010
     d0  0000000000000000  d1  0000000000000000
     d2  0000000000000000  d3  0000000000000000
     d4  0000000000000000  d5  0000000000000000
     d6  0000000043200000  d7  0000000000000000
     d8  0000000000000000  d9  440b000043150000
     d10 000000000000022c  d11 0000000000000000
     d12 0000000000000000  d13 0000000000000000
     d14 0000000000000000  d15 0000000000000000
     d16 0000000040aa0eb8  d17 3f80000041808889
     d18 3f80000041c4cccd  d19 0701070100700798
     d20 0000000000000c07  d21 0000043f00890000
     d22 0000000000080008  d23 0000000000000008
     d24 0078007700760074  d25 007a0078007a0079
     d26 0000000000000000  d27 0000000000000000
     d28 0048004700460044  d29 004a0048004a0049
     d30 007a007a007a007a  d31 0000000000000000
     scr 80000017
    
             #00  pc 0000d8d0  /system/lib/libc.so (kill)
             #01  pc 000660dc  /system/lib/libandroid_runtime.so (_Z29android_os_Process_sendSignalP7_JNIEnvP8_jobjectii)
    
    code around pc:
    400bd8b0 e2601000 e0100001 116f0f10 12600020  ..`.......o. .`.
    400bd8c0 e12fff1e e92d50f0 e3a07025 ef000000  ../..P-.%p......
    400bd8d0 e8bd50f0 e1b00000 512fff1e ea00b264  .P......../Qd...
    400bd8e0 e92d50f0 e3a070ee ef000000 e8bd50f0  .P-..p.......P..
    400bd8f0 e1b00000 512fff1e ea00b25d f5d0f000  ....../Q].......
    
    code around lr:
    401db0bc 0003ecaa b5102a00 4610dd03 f7d34619  .....*.....F.F..
    401db0cc bd10e886 b5102a00 4610dd03 f7d34619  .....*.....F.F..
    401db0dc bd10e87e 4601b513 4611b91a e820f7d1  ~......F...F.. .
    401db0ec ac01e00c f7fe4620 9b01fce7 681ab133  .... F......3..h
    401db0fc f8524621 18180c0c eebcf7d0 bf00bd1c  !FR.............
    
    stack:
        be895580  40aa0eb8  /dev/ashmem/dalvik-heap (deleted)
        be895584  be8955c8  [stack]
        be895588  0095d5c8  [heap]
        be89558c  be8955c8  [stack]
        be895590  a5c1c5ac  
        be895594  40890bd7  /system/lib/libdvm.so
        be895598  00c1ce30  [heap]
        be89559c  400f85a0  
        be8955a0  00000028  
        be8955a4  400f8554  
        be8955a8  00c1ce38  [heap]
        be8955ac  00000006  
        be8955b0  4badece0  
        be8955b4  400c5bfd  /system/lib/libc.so
        be8955b8  df0027ad  
        be8955bc  00000000  
    #01 be8955c0  4bbe49d8  /dev/ashmem/dalvik-LinearAlloc (deleted)
        be8955c4  00958e38  [heap]
        be8955c8  00000000  
        be8955cc  4badeda4  
        be8955d0  4021ae3c  /system/lib/libandroid_runtime.so
        be8955d4  401db0df  /system/lib/libandroid_runtime.so
        be8955d8  4bbe49d8  /dev/ashmem/dalvik-LinearAlloc (deleted)
        be8955dc  4085f134  /system/lib/libdvm.so
        be8955e0  4baded9c  
        be8955e4  00000001  
        be8955e8  40aa0eb8  /dev/ashmem/dalvik-heap (deleted)
        be8955ec  00958e48  [heap]
        be8955f0  00000002  
        be8955f4  40899299  /system/lib/libdvm.so
        be8955f8  4baded9c  
        be8955fc  4d01bc75  /data/dalvik-cache/system@framework@framework.jar@classes.dex
        be895600  401db0d1  /system/lib/libandroid_runtime.so
        be895604  00958e48  [heap]
    --------- tail end of log /dev/log/main
    06-14 13:05:39.319  1373  1373 D ActivityThread: setTargetHeapUtilization:0.25
    06-14 13:05:39.329  1373  1373 D ActivityThread: setTargetHeapIdealFree:8388608
    06-14 13:05:39.329  1373  1373 D ActivityThread: setTargetHeapConcurrentStart:2097152
    06-14 13:05:54.269  1373  1373 I BadBehaviorActivity: Native crash pressed -- about to kill -11 self
    06-14 13:05:54.269  1373  1373 F libc    : Fatal signal 11 (SIGSEGV) at 0x0000055d (code=0)
    --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --


### Effects

If a crash happens in the user space application, e.g., market applications or system-privileged OEM application, it kills the process and restarts if it is necessary.

If a crash happens in system_server, it restarts Android user space.

## Tombstone如何解析？

Use the following toolchain tools to map the failed address to the source code:

- arm-eabi-objdump
- arm-eabi-addr2line


1. Android 产生出來的还沒進行strip的执行档或shared libraries 是放在

    out/target/product/YOUR_PRODUCT_NAME/symbols/system/bin
    out/target/product/YOUR_PRODUCT_NAME/symbols/system/lib

2. Android所使用的toolchain是放在

    prebuilt/linux-x86/toolchain/arm-eabi-4.4.3/bin/

3. 假设我們要看 #00  pc 0000d8d0  /system/lib/libc.so (kill)
             #01  pc 000660dc  /system/lib/libandroid_runtime.so  是 call 到哪一個function
 (假设tombstone_XX 是在 $android_root 目录下)

    $ ./prebuilt/linux-x86/toolchain/arm-eabi-4.4.3/bin/arm-eabi-addr2line -f -e ./out/target/product/YOUR_PRODUCT_NAME/symbols/system/lib/libc.so  0000d8d0 
      
    memcmp 
   ...bionic/libc/arch-arm/bionic/memcmp.S:131

## Reference

*[android linker 浅析](http://blog.csdn.net/challen537/article/details/6621488)
