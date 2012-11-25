---
layout: master
title: SWP LDREX and STREX
---



## Overview

## SWP

早期的ARM指令集（V6前）提供SWP指令，该指令可原子交换寄存器和内存数据，用于实现信号量操作。
 
例子：
 
	sem_wati:
	MOV R1,#0   
	LDR R0,=SEM
	SWP R1,R1,[R0] ;取出信号量，并设置其为0
	CMP R1,#0 ;判断是否有信号
	BEQ sem_wait ;若没有信号，则等待

SWP指令的缺点是会lock总线，影响系统性能。

## LDREX & STREX

新的ARM指令（V6、V7）采用LDREX和STREX指令替换了SWP指令，可以实现对共享内存的非阻塞同步。
 
###指令格式

	LDREX <Rt>,[Rn]
	STREX <Rd>,<Rt>,[Rn] ；STREX成功，Rd置0


	LDREX{cond} Rt, [Rn {, #offset}]
	STREX{cond} Rd, Rt, [Rn {, #offset}]
	LDREXB{cond} Rt, [Rn]
	STREXB{cond} Rd, Rt, [Rn]
	LDREXH{cond} Rt, [Rn]
	STREXH{cond} Rd, Rt, [Rn]
	LDREXD{cond} Rt, Rt2, [Rn]
	STREXD{cond} Rd, Rt, Rt2, [Rn]

ARM LDREX and STREX are available in ARMv6 and above.

ARM LDREXB, LDREXH, LDREXD, STREXB, STREXD, and STREXH are available in ARMv6K and above.

All these 32-bit Thumb instructions are available in ARMv6T2 and above, except that LDREXD and STREXD are not available in the ARMv7-M profile.

There are no 16-bit versions of these instructions.

###LDREX：


LDREX loads data from memory.

If the physical address has the Shared TLB attribute, LDREX tags the physical address as exclusive access for the current processor, and clears any exclusive access tag for this processor for any other physical address.

Otherwise, it tags the fact that the executing processor has an outstanding tagged physical address.


###STREX：


STREX performs a conditional store to memory. The conditions are as follows:

If the physical address does not have the Shared TLB attribute, and the executing processor has an outstanding tagged physical address, the store takes place, the tag is cleared, and the value 0 is returned in Rd.

If the physical address does not have the Shared TLB attribute, and the executing processor does not have an outstanding tagged physical address, the store does not take place, and the value 1 is returned in Rd.

If the physical address has the Shared TLB attribute, and the physical address is tagged as exclusive access for the executing processor, the store takes place, the tag is cleared, and the value 0 is returned in Rd.

If the physical address has the Shared TLB attribute, and the physical address is not tagged as exclusive access for the executing processor, the store does not take place, and the value 1 is returned in Rd.

 

###例子:

	lock_mutex
	LDREX r1,[r0]    ; 检查是否lock
	CMP   r1,#LOCK   ; 和LOCK比较，LOCK是0
	BEQ   lock_mutex ; 相等说明被锁定，自旋
 
	MOV   r1,#LOCK   ; 不相等，加锁
	STREX r2,r1,[r0] ; 尝试将r1写入锁
	CMP   r2,#0x0    ; 判断是否加锁成功（可能出现竞争导致加锁失败）
	BNE   lock_mutex ; 如果不成功，从头判断
	DMB              ; 内存屏障保证前面操作成功
	BX    lr         ; 返回
 
 
	ulock_mutex
	DMB                ; 内存屏障，保证安全访问
	MOV   r1,#UNLOCKED ; 解锁
	STR   r1,[r0]      ;
	BX    lr           ;
 
 
LDREX和STREX是通过ARM内核的一个叫**Exclusive Monitor**的机制实现的，EM是一个状态机。

	LDREX指令将Monitor置为Exclusive状态
	STREX指令将Exclusive状态置回为Open状态，由此保证访问的唯一性。

但是在进程切换时，可能导致EM被打乱，因此需要执行CLREX指令，清除Exclusive Monitor。

## Source&Reference


