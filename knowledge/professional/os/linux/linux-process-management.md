---
layout: master
title: Linux Process management
---

## Overview

Linux 2.4内核以及之前的版本都不支持线程.2.6内核中有thread，但仍不是线程

Linux中的线程是在用户态实现的，但Linux内核对用户态线程有一定的辅助支持

## 进程描述符

源码include/linux/sched.h定义

    struct task_struct

基本信息

管理信息

控制信息

## 进程标识

使用进程描述符地址：进程和进程描述符之间有非常严格的一一对应关系，使得用32位进程描述符地址标识进程非常方便

使用PID (Process ID，PID)： 每个进程的PID都存放在进程描述符的pid域中

	pid_t pid;
	pid_t tgid;

    typedef __kernel_pid_t		pid_t;
    typedef int			__kernel_pid_t;

###最大进程号

int pid_max = PID_MAX_DEFAULT;

/*
 * This controls the default maximum pid allocated to a process
 */
    #define PID_MAX_DEFAULT (CONFIG_BASE_SMALL ? 0x1000 : 0x8000)


### Pid的管理和分配

 do_fork->copy_process->alloc_pid



Current宏的使用

Current宏可以看成当前进程的进程描述符指针，在内核中直接使用

比如current->pid返回在CPU上正在执行的进程的PID


## 进程状态

----------
- > /*
- >  * Task state bitmask. NOTE! These bits are also
- >  * encoded in fs/proc/array.c: get_task_state().
- >  *
- >  * We have two separate sets of flags: task->state
- >  * is about runnability, while task->exit_state are
- >  * about the task exiting. Confusing, but this way
- >  * modifying one set can't modify the other one by
- >  * mistake.
- >  */
- > #define TASK_RUNNING		0
- > #define TASK_INTERRUPTIBLE	1
- > #define TASK_UNINTERRUPTIBLE	2
- > #define __TASK_STOPPED		4
- > #define __TASK_TRACED		8
- > /* in tsk->exit_state */
- > #define EXIT_ZOMBIE		16
- > #define EXIT_DEAD		32
- > /* in tsk->state again */
- > #define TASK_DEAD		64
- > #define TASK_WAKEKILL		128
- > #define TASK_WAKING		256
- > #define TASK_STATE_MAX		512


### 状态转换

TASK->(fork)->TASK_RUNNING【就绪】->(schedule)->TASK_RUNNING【运行】->(do_exit)->TASK_DEAD/EXIT_ZOMBIE/EXIT_DEAD

TASK_RUNNING【运行】->(sleep/wait)->TASK_INTERRUPTIBLE/TASK_UNINTERRUPTIBLE->(ready)->TASK_RUNNING【就绪】

TASK_RUNNING【运行】->(抢占)->TASK_RUNNING【就绪】

### 获得一个进程的pid

系统调用getpid: this returns the tgid not the pid.


## 进程和进程的内核堆栈

Linux为每个进程分配一个8KB大小的内存区域，用于存放该进程两个不同的数据结构：

- Thread_info
- 进程的内核堆栈

include/linux/sched.h

     union thread_union {
	 struct thread_info thread_info;
	 unsigned long stack[THREAD_SIZE/sizeof(long)];
     };

thread_info由体系结构相关部分定义



进程处于内核态时使用，不同于用户态堆栈

内核控制路径所用的堆栈很少，因此对栈和Thread_info来说，8KB足够了


## 进程链表

为了对给定类型的进程(比如所有在可运行状态下的进程)进行有效的搜索，内核维护了几个进程链表

所有进程链表

struct task_struct {

	struct list_head tasks;



struct list_head {
	struct list_head *next, *prev;
};


### 进程链表中的插入和删除

参见include/linux/list.h或者lib/list_debug.c

例如，在do_fork调用的copy_process中


### TASK_RUNNING状态的进程组织

struct task_struct {

	const struct sched_class *sched_class;//调度类


fair_sched_class

idle_sched_class


static const struct sched_class rt_sched_class;

每个cpu有一个运行队列


## Reference

