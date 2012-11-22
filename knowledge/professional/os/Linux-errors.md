---
layout: master
title: Linux errors
---

## Overview

1,spinlock
2,gic interrupt handling processdure
3,IPI

### BUG: spinlock bad magic on CPU#1, swapper/1/0

> [    4.636836] [(1980-01-07 18:29:45.308783335 UTC)] BUG: spinlock bad magic on CPU#1, swapper/1/0
> [    4.637133] [(1980-01-07 18:29:45.309078335 UTC)]  lock: irq_controller_lock, rawlock 0x1, .magic: dead4ead, .owner: swapper/1/0, .owner_cpu: 1
> [    4.642224] [(1980-01-07 18:29:45.314170001 UTC)] [<c001523c>] (unwind_backtrace+0x0/0x12c) from [<c027fda0>] (do_raw_spin_unlock+0x20/0xc0)
> [    4.654788] [(1980-01-07 18:29:45.326733334 UTC)] [<c027fda0>] (do_raw_spin_unlock+0x20/0xc0) from [<c05e9478>] (_raw_spin_unlock+0x8/0x34)
> [    4.667283] [(1980-01-07 18:29:45.339228334 UTC)] [<c05e9478>] (_raw_spin_unlock+0x8/0x34) from [<c00084bc>] (gic_handle_irq+0x48/0xbc)
> [    4.679434] [(1980-01-07 18:29:45.351378333 UTC)] [<c00084bc>] (gic_handle_irq+0x48/0xbc) from [<c05e9a80>] (__irq_svc+0x40/0x70)


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




### Exception stack(0xd8e65f48 to 0xd8e65f90)



###  Internal error: Oops - undefined instruction: 0 [#1] PREEMPT SMP ARM

> [    4.779729] [(1980-01-07 18:29:45.451674998 UTC)] Internal error: Oops - undefined instruction: 0 [#1] PREEMPT SMP ARM
> [    4.788453] [(1980-01-07 18:29:45.460398332 UTC)] Modules linked in:
> [    4.794793] [(1980-01-07 18:29:45.466736666 UTC)] CPU: 0    Not tainted  (3.4.0-svn7077 #5)
> [    4.803131] [(1980-01-07 18:29:45.475076667 UTC)] PC is at tick_check_idle+0x1a0/0x1ac
> [    4.811026] [(1980-01-07 18:29:45.482970002 UTC)] LR is at update_ts_time_stats+0x38/0x9c
> [    4.819186] [(1980-01-07 18:29:45.491130003 UTC)] pc : [<c00d6304>]    lr : [<c00d6100>]    psr: 60000193
> [    4.819199] [(1980-01-07 18:29:45.491143336 UTC)] sp : c08adea0  ip : 0000000b  fp : c08adecc
> [    4.837238] [(1980-01-07 18:29:45.509183338 UTC)] r10: 00000403  r9 : c0b00108  r8 : 00000000
> [    4.845746] [(1980-01-07 18:29:45.517691673 UTC)] r7 : c1e91b50  r6 : 1d49a012  r5 : 00000001  r4 : c1e91b50
> [    4.855553] [(1980-01-07 18:29:45.527498340 UTC)] r3 : 00000000  r2 : c1e91b50  r1 : 00000000  r0 : 00000000
> [    4.865364] [(1980-01-07 18:29:45.537310008 UTC)] Flags: nZCv  IRQs off  FIQs on  Mode SVC_32  ISA ARM  Segment kernel
> [    4.876039] [(1980-01-07 18:29:45.547985009 UTC)] Control: 10c5387d  Table: 2848804a  DAC: 00000015
> [    4.885066] [(1980-01-07 18:29:45.557011677 UTC)] 
> [    4.885076] [(1980-01-07 18:29:45.557021677 UTC)] PC: 0xc00d6284:
> [    4.895916] [(1980-01-07 18:29:45.567860012 UTC)] 6284  e1a02000 e5930000 e58dc010 e2000401 e58d2014 e3500401 1a000015 e3a02003
> [    4.907374] [(1980-01-07 18:29:45.579320013 UTC)] 62a4  e28d3010 e3a0c008 e58dc000 ebfee34a ea00000f f10c0080 e1a0100f e1a0c00e
> [    4.918833] [(1980-01-07 18:29:45.590776681 UTC)] 62c4  e1a02000 e5930000 e58dc010 e2000401 e58d2014 e3500401 1a000004 e3002103
> [    4.930289] [(1980-01-07 18:29:45.602235016 UTC)] 62e4  e28d3010 e3a0c008 e58dc000 ebfee33a e121f004 eb004b44 e28dd018 e8bd81f0
> [    4.941749] [(1980-01-07 18:29:45.613695017 UTC)] 6304  c08d4114 c08a7b50 c0b669a4 e59f20d0 e92d43f0 e59f30cc e5922000 e24dd01c
> [    4.953208] [(1980-01-07 18:29:45.625153352 UTC)] 6324  e3520000 e59f20c0 e1a06000 e1a05001 03e03000 e7924100 03e02000 0a000024
> [    4.964666] [(1980-01-07 18:29:45.636610020 UTC)] 6344  e28d0008 e0834004 ebffe911 e3550000 e1cd80d8 0a000006 e1a00006 e1a02008
> [    4.976124] [(1980-01-07 18:29:45.648070022 UTC)] 6364  e1a03009 e1a01004 e58d5000 ebffff54 ea00000d e594306c e3530000 0a00000a

./arch/arm/kernel/entry-armv.S

	__und_svc

./arch/arm/kernel/traps.c

	do_undefinstr
		arm_notify_die("Oops - undefined instruction", regs, &info, 0, 6);
			void die(const char *str, struct pt_regs *regs, int err)
				__die
					printk(KERN_EMERG "Internal error: %s: %x [#%d]" S_PREEMPT S_SM
					print_modules();
					__show_regs(regs);
					printk(KERN_EMERG "Process %.*s (pid: %d, stack limit = 0x%p)\n",
					dump_mem(KERN_EMERG, "Stack: ", regs->ARM_sp
					dump_backtrace(regs, tsk);
						unwind_backtrace
							dump_backtrace_entry
			oops_exit();


./kernel/panic.c

	oops_exit
	print_oops_end_marker
		printk(KERN_WARNING "---[ end trace %016llx ]---\n",

## Reference

