---
layout: master
title: UNIX Signal
---

##1 Overview

**A signal** is a small message that notifies a process that an event of some type has occurred in the system.

- Kernel abstraction for exceptions and interrupts.每种信号类型都对应某个类型的系统事件
- Sent from the kernel (sometimes at the request of another process) to a process.
- Different signals are identified by small integer ID’s (1-30)
- The only information in a signal is its ID and the fact that it arrived.

##2 Signal Concepts 

###2.1 Sending a signal

**2.1.1 What** 

Kernel sends (delivers) a signal to a destination process by updating some state in the *context* of the destination process.

**2.1.2 Why**
 
Kernel sends a signal for one of the following reasons:

- Kernel has detected a system event such as divide-by-zero (SIGFPE) or the termination of a child process (SIGCHLD)
- Another process has invoked the kill system call to explicitly request the kernel to send a signal to the destination process.

**2.1.3 How**

**2.1.3.1 Sending Signals with kill Program**

kill program sends arbitrary signal to a process or process group

**2.1.3.2 Sending Signals from the Keyboard**

Typing ctrl-c (ctrl-z) sends a SIGINT (SIGTSTP) to every job in the foreground process group.

- SIGINT – default action is to terminate each process 
- SIGTSTP – default action is to stop (suspend) each process

**2.1.3.3 Sending Signals with kill Function**

int kill(pid_t pid, int sig);

**2.1.3.4 Sending Signals to itself with alarm Function**

unsigned int alarm(unsigned int secs);


###2.2 Receiving a signal

**2.2.1 What**

A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal.


Three possible ways to react:

- Ignore the signal (do nothing)
- Terminate the process (with optional core dump).
- Catch the signal by executing a user-level function called a **signal handler.**
Akin to a hardware exception handler being called in response to an asynchronous interrupt.

**2.2.2 How**

Suppose  kernel is returning from an exception handler and is ready to pass control to process p.

Kernel computes pnb = pending & ~blocked

- The set of pending nonblocked signals for process p 

If  (pnb == 0) 

- Pass control to next instruction in the logical flow for p.

Else

- Choose least nonzero bit k in pnb and force process p to receive signal k.
- The receipt of the signal triggers **some action** by p
- Repeat for all nonzero k in pnb.
- Pass control to next instruction in logical flow for p.

**2.2.2.1 default action**

Each signal type has a predefined default action, which is one of:

- The process terminates
- The process terminates and dumps core.
- The process stops until restarted by a SIGCONT signal.
- The process ignores the signal.

***For SIGKILL and SIGSTOP the action is "always", not just "default".***


**2.2.2.2 Installing Signal Handlers**

The signal function modifies the default action associated with the receipt of signal signum:

    handler_t *signal(int signum, handler_t *handler)

Different values for handler:

- SIG_IGN: ignore signals of type signum
- SIG_DFL: revert to the default action on receipt of signals of type signum.
- Otherwise, handler is the address of a signal handler

    - Called when process receives signal of type signum
    - Referred to as “installing” the handler.
    - Executing handler is called “catching” or “handling” the signal.
    When the handler executes its return statement, control passes back to instruction in the control flow of the process that was interrupted by receipt of the signal.


###2.3 pending signal

A signal is pending if it has been sent but not yet received.

There can be at most one pending signal of any particular type.

Important: Signals are not queued

- If a process has a pending signal of type k, then subsequent signals of type k that are sent to that process are discarded.

A process can block the receipt of certain signals.

- Blocked signals can be delivered, but will not be received until the signal is unblocked.

A pending signal is received at most once.

**How**

Kernel maintains **pending and blocked bit vectors** in the context of each process.

pending – represents the set of pending signals

- Kernel sets bit k in pending whenever a signal of type k is delivered.
- Kernel clears bit k in pending whenever a signal of type k is received 

blocked – represents the set of blocked signals

- Can be set and cleared by the application using the sigprocmask function.


## Signal handling problems.

### Pending signals are not queued

For each signal type, just have single bit indicating whether or not signal is pending

Even if multiple processes have sent this signal

### Living With Nonqueuing Signals

### Signal arrival during long system calls (say a read)

Signal handler interrupts read() call

Linux: upon return from signal handler, the read() call is restarted automatically

Some other flavors of Unix can cause the read() call to fail with an EINTER error number (errno) in this case, the application program can restart the slow system call

Subtle differences like these complicate the writing of portable code that uses signals.

## UNIX signal 集合

    信号名字    BSD Linux Mac Solaris  默认动作    说明
    SIGABRT    x     x   x     x     终止+core  异常终止(abort)
    SIGALRM    x     x   x     x     终止       闹钟超时(alarm)
    SIGBUS     x     x   x     x     终止+core  总线错误
    SIGCANCEL                  x     忽略       线程库内部使用
    SIGCHLD    x     x   x     x     忽略       子进程暂停或终止
    SIGCONT    x     x   x     x     忽略       通知已暂停进程继续
    SIGEMT     x     x   x     x     终止+core  来自PDP-11之(emulator trap)指令
    SIGFPE     x     x   x     x     终止+core  算术异常
    SIGFREEZE                  x     忽略       检查点冻结
    SIGHUP     x     x   x     x     终止       拨号连接被挂断，通常由会话首进程接收
    SIGILL     x     x   x     x     终止+core  非法硬件指令
    SIGINFO    x         x           忽略       C-T产生
    SIGINT     x     x   x     x     终止       DELETE或C-C产生
    SIGIO      x     x   x     x     终止/忽略   异步I/O
    SIGIOT     x     x   x     x     终止+core  PDP-11之指令，四个系统将其值定义为SIGABRT
    SIGKILL    x     x   x     x     终止       **终止，无法忽略和捕捉**
    SIGLWP                     x     忽略       线程库内部使用
    SIGPIPE    x     x   x     x     终止       写管道或SOCK_STREAM时接收数据进程已经终止
    SIGPOLL          x         x     终止       轮询事件，俩系统将其等同于SIGIO
    SIGPROF    x     x   x     x     终止       setitimer设置的梗概统计间隔计时器到期
    SIGPWR           x         x     终止/忽略   如果存在电池，则当电量低时警告
    SIGQUIT    x     x   x     x     终止+core  C-\产生
    SIGSEGV    x     x   x     x     终止+core  内存段访问异常
    SIGSTKFLT        x               终止       协处理器故障，早期Linux定义信号
    SIGSTOP    x     x   x     x     暂停       **暂停，无法忽略和捕捉**
    SIGSYS     x     x   x     x     终止+core  无效系统调用
    SIGTERM    x     x   x     x     终止       kill发送之默认终止信号
    SIGTHAW                    x     忽略       检查点解冻
    SIGTRAP    x     x   x     x     终止+core  PDP-11的TRAP指令
    SIGTSTP    x     x   x     x     暂停       C-Z产生
    SIGTTIN    x     x   x     x     暂停       后台读控制tty
    SIGTTOU    x     x   x     x     暂停       后台写控制tty
    SIGURG     x     x   x     x     忽略       紧急情况（套接字）
    SIGUSR1    x     x   x     x     终止       用户定义信号
    SIGUSR2    x     x   x     x     终止       用户定义信号
    SIGVALRM   x     x   x     x     终止       虚拟时间闹钟（setitimer）
    SIGWAITING                 x     忽略       线程库内部使用
    SIGWINCH   x     x   x     x     忽略       终端窗口大小改变
    SIGXCPU    x     x   x     x     终止+core  超过CPU资源限制（setrlimit）
    SIGXFSZ    x     x   x     x     终止+core  超过文件长度限制（setrlimit）
    SIGXRES                    x     忽略       超过资源限制


![](http://hi.csdn.net/attachment/201111/21/0_1321885982nxIp.gif)
