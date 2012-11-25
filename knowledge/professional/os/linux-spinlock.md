---
layout: master
title: Linux spinlock
---

##1 Overview



##2 spin locks

###2.1 simple spinlock

Codes

		static DEFINE_SPINLOCK(xxx_lock);

        unsigned long flags;

        spin_lock_irqsave(&xxx_lock, flags);
        ... critical section here ..
        spin_unlock_irqrestore(&xxx_lock, flags);

The above is always safe. It will disable interrupts _locally_, but the
spinlock itself will guarantee the global lock, so it will guarantee that
there is **only one thread-of-control** within the region(s) protected by that
lock. This works well even under UP also, so the code does _not_ need to
worry about **UP** vs **SMP** issues: the spinlocks work correctly under both.


The single spin-lock primitives above are by no means the only ones. They
are **the most safe ones**, and the ones that **work under all circumstances**,
but partly _because_ they are safe they are also fairly **slow**. They are slower
than they'd need to be, because they do have to **disable interrupts**
(which is just a single instruction on a x86, but it's an expensive one -
and on other architectures it can be worse).


###2.2 reader-writer spinlocks

If your data accesses have a very natural pattern where you usually tend
to **mostly read** from the shared variables, the reader-writer locks
(rw_lock) versions of the spinlocks are sometimes useful. They allow multiple
readers to be in the same critical region at once, but if somebody wants
to change the variables it has to get an exclusive write lock.

routines

   		rwlock_t xxx_lock = __RW_LOCK_UNLOCKED(xxx_lock);

        unsigned long flags;

        read_lock_irqsave(&xxx_lock, flags);
        .. critical section that only reads the info ...
        read_unlock_irqrestore(&xxx_lock, flags);

        write_lock_irqsave(&xxx_lock, flags);
        .. read and write exclusive access to the info ...
        write_unlock_irqrestore(&xxx_lock, flags);

The above kind of lock may be useful for complex data structures like
linked lists, especially searching for entries without changing the list
itself.  The read lock allows many concurrent readers.  Anything that
_changes_ the list will have to get the write lock.





## Spin lock API
The spin-lock is safe only when you _also_ use the lock itself to do locking across CPU's, which implies that EVERYTHING that touches a shared variable has to agree about the spinlock they want to use.

- spin_lock_init(lock) 一个自旋锁时可使用接口函数将其初始化为锁定状态 
- spin_lock(lock) 用于锁定自旋锁如果成功则返回否则循环等待自旋锁变为空闲
- spin_unlock(lock) 释放自旋锁lock重新设置自旋锁为锁定状态 
- spin_is_locked(lock) 判断当前自旋锁是否处于锁定状态 
- spin_unlock_wait(lock) 循环等待、直到自旋锁lock变为可用状态 
- spin_trylock(lock) 尝试锁定自旋锁lock如不成功则返回0否则锁定并返1
- spin_can_lock(lock) 判断自旋锁lock是否处于空闲状态

## 排队自旋锁


排队自旋锁(FIFO Ticket Spinlock)是 Linux 内核 2.6.25 版本中引入的一种新型自旋锁，它解决了传统自旋锁由于无序竞争导致的“公平性”问题。


LINUX 2.6.35版本将spin lock实现更改为 ticket lock.

spin_lock数据结构除了用于内核调试之外字段为raw\_spinlock rlock。 
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


## SPIN LOCK Debug

### config

CONFIG_DEBUG_SPINLOCK


### code


	raw_spin_unlock
	_raw_spin_unlock

kernel/lib/spinlock_debug.c


	do_raw_spin_unlock->debug_spin_unlock
					  ->arch_spin_unlock
	debug_spin_unlock->
	SPIN_BUG_ON
	spin_bug
	spin_dump
		printk(KERN_EMERG "BUG: spinlock %s on CPU#%d, %s/%d\n"
		printk(KERN_EMERG " lock: %ps, .magic: %08x, .owner: %s/%d,
		dump_stack

### debug\_spin_unlock

#### SPIN\_BUG_ON(lock->magic != SPINLOCK\_MAGIC, lock, "bad magic");



	SPIN_BUG_ON(!raw_spin_is_locked(lock), lock, "already unlocked");
	SPIN_BUG_ON(lock->owner != current, lock, "wrong owner");
	SPIN_BUG_ON(lock->owner_cpu != raw_smp_processor_id(),
							lock, "wrong CPU");

### debug\_spin\_lock\_before(raw\_spinlock\_t *lock)

        SPIN_BUG_ON(lock->magic != SPINLOCK_MAGIC, lock, "bad magic");
        SPIN_BUG_ON(lock->owner == current, lock, "recursion");
        SPIN_BUG_ON(lock->owner_cpu == raw_smp_processor_id(), lock, "cpu recursion");



## Reference

[Ticket spinlocks](http://lwn.net/Articles/267968/)
