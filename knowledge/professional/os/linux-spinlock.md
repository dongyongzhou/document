---
layout: master
title: Linux spinlock
---

## Overview

The spin-lock is safe only when you _also_ use the lock itself to do locking across CPU's, which implies that EVERYTHING that touches a shared variable has to agree about the spinlock they want to use.


- spin_lock_init(lock) 一个自旋锁时可使用接口函数将其初始化为锁定状态 
- spin_lock(lock) 用于锁定自旋锁如果成功则返回否则循环等待自旋锁变为空闲
- spin_unlock(lock) 释放自旋锁lock重新设置自旋锁为锁定状态 
- spin_is_locked(lock) 判断当前自旋锁是否处于锁定状态 
- spin_unlock_wait(lock) 循环等待、直到自旋锁lock变为可用状态 
- spin_trylock(lock) 尝试锁定自旋锁lock如不成功则返回0否则锁定并返1
- spin_can_lock(lock) 判断自旋锁lock是否处于空闲状态



排队自旋锁(FIFO Ticket Spinlock)是 Linux 内核 2.6.25 版本中引入的一种新型自旋锁，它解决了传统自旋锁由于无序竞争导致的“公平性”问题。


LINUX 2.6.35版本将spin lock实现更改为 ticket lock.

spin_lock数据结构除了用于内核调试之外字段为raw_spinlock rlock。 
ticket spinlock将rlock字段分解为如下两部分  
Next是下一个票号
Owner是允许使用自旋锁的票号。

加锁时CPU取Next并将rlock.Next + 1。将Next与Owner相比较
若相同
则加锁成功
否则循环等待、直到Next = rlock.Owner为止。

解锁则直接将Owner + 1即可。



表 1. 排队自旋锁 API

		宏	底层实现函数	描述

	spin_lock_init	无	将锁置为初始未使用状态(值为 0)
	spin_lock	__raw_spin_lock	忙等待直到 Owner 域等于本地票据序号
	spin_unlock	__raw_spin_unlock	Owner 域加 1，将锁传给后续等待线程
	spin_unlock_wait	__raw_spin_unlock_wait	不申请锁，忙等待直到锁处于未使用状态
	spin_is_locked	__raw_spin_is_locked	测试锁是否处于使用状态
	spin_trylock	__raw_spin_trylock	如果锁处于未使用状态，获得锁；否则直接返回



LDREX和STREX指令的应用。

DREX和STREX指令是在V6以后才出现的，代替了V6以前的swp指令。可以让bus监控LDREX和STREX指令之间有无其它CPU和DMA来存取过这个地址，若有的话STREX指令的第一个寄存器里设置为1（动作失败），若没有，指令的第一个寄存器里设置为0（动作成功）。


LDREX和STREX是通过ARM内核的一个叫Exclusive Monitor的机制实现的，EM是一个状态机。
LDREX指令将Monitor置为Exclusive状态，STREX指令将Exclusive状态置回为Open状态，由此保证访问的唯一性。
但是在进程切换时，可能导致EM被打乱，因此需要执行CLREX指令，清除Exclusive Monitor。

## Reference

[Ticket spinlocks](http://lwn.net/Articles/267968/)
