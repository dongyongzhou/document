---
layout: master
title: Root android device
---

## Goal
- gaining super user privileges
- has full control over the operating system.

## Buffer overflow attacking: Inject malicious code

### get malicious code into the memory.

a. inject the malicious code into the buffer(stack), depend on the size

### How to jump to the malicious code

set the return address to the malicious code

### find the address of the injected malicious code

trying..trying many times...

a.to make it succeed easily, we could debug it in personal computer.

- the stack space is always starting from a fixed address.
- the stack will not be that deep. 

b.make multiple entrance available

- fill a lot of NOP into the buffer.It will run finally to our code.

### What code do we write

- Shell code, Binary finally

		char *name[2];
		name[0]="/bin/sh";
		name[1]=NULL;
		execve(name[0],name,NULL);

- challenge one: use stack. NO parameter
- challenge two: loader partner, no loader, no dependence on loader.
- challenge three: no zero could be used.

### buffer overflow countermeasure

**Method one: Address Space Layout Randomization (ASLR)**

make it harder to find the entry address of the malicious code.

disable ASLR

check if ASLR is working or not(Position independent code must be set), use: hardening-check executable_name


有一个内核参数randomize_va_space用于控制系统级ASLR:

	0: 关闭ASLR
	1: mmap base、stack、vdso page将随机化。这意味着.so文件将被加载到随机地址。链接时指定了-pie选项的可执行程序，其代码段加载地址将被随机化。配置内核时如果指定了CONFIG_COMPAT_BRK，randomize_va_space缺省为1。此时heap没有随机化。
	2: 在1的基础上增加了heap随机化。配置内核时如果禁用CONFIG_COMPAT_BRK，randomize_va_space缺省为2。


Disable ASLR:

		sudo bash -c 'echo 0 > /proc/sys/kernel/randomize_va_space'

Kernel config

		kernel/init/Kconfig
		1685config COMPAT_BRK
		1686	bool "Disable heap randomization"
		1687	default y
		1688	help
		1689	  Randomizing heap placement makes heap exploits harder, but it
		1690	  also breaks ancient binaries (including anything libc5 based).
		1691	  This option changes the bootup default to heap randomization
		1692	  disabled, and can be overridden at runtime by setting
		1693	  /proc/sys/kernel/randomize_va_space to 2.

**Method two: Data Execution Prevention (DEP)**

a, mark stack (page)as not executable.
program

- executable stack: gcc -z execstack -o test test.c
- non-executable stack: gcc -z noexecstack -o test test.c

1. Make stack space executable

		/kernel/tools/perf/config/Makefile
		
		149# Enforce a non-executable stack, as we may regress (again) in the future by
		150# adding assembler files missing the .GNU-stack linker note.
		151LDFLAGS += -Wl,-z,noexecstack------------->execstack




**Method three:detect buffer overflow**

a, boundery check of course by programmer

b, **Stack Canaries**: add a magic number(random number) between RET and SP address,  push Random number into heap or static buffer, supported by GCC compiler.

GCC: Stack-smashing Protection（SSP) or called ProPolice

Stack Guard，Stack-smashing Protection(SSP)

how to turn it off?/Disable canaries 

	-fstack-protector：启用堆栈保护，不过只为局部变量中含有 char 数组的函数插入保护代码。
	
	-fstack-protector-all：启用堆栈保护，为所有函数插入保护代码。
	
	-fno-stack-protector：禁用堆栈保护。

	Example: # gcc -o stack -fno-stack-protector -z noexecstack stack.c

kernel

		/kernel/tools/perf/config/Makefile
		
		265ifeq ($(feature-stackprotector-all), 1)
		266  CFLAGS += -fstack-protector-all---------------->fno-stack-protector
		267endif


**Method four:return address can not be compromised, place the return address below.**

a, stack shadow , backup stack and compared.


## Buffer overflow attacking Two: Return to libc attack

as stack is not executable at all. we could use existing code.

a, jump to system function.

## Analysis

ROOT: The basic idea is putting binary "su" under /system/bin/ or /system/xbin/


Impact


gaining super user privileges

## Hacking

### Preparation


How to check?

	[ustc@zdyhost ~]$ ps -ef |grep bufbomb
	ustc     10532 10481  0 11:52 pts/2    00:00:00 gdb bufbomb
	ustc     10545 10532  0 11:52 pts/2    00:00:00 /home/ustc/course/csapp/exp3/bufbomb -t SA10225501+SA10225129
	ustc     10650 10604  0 11:55 pts/3    00:00:00 grep bufbomb
	[ustc@zdyhost ~]$ cat /proc/10545/maps



Debugging 

	B::sys.m.d
	B::sys.m.a
	B::b
	B::Data.LOAD.elf  \\xxx\out\target\product\<product>\obj\KERNEL_OBJ\vmlinux /nocode /noclear
	B::b.s set_image_version /o
	B::mmu.list


How to build kernel/boot.img seperately.

	make -C kernel O=../out/target/product/<target>/obj/KERNEL_OBJ ARCH=arm64 CROSS_COMPILE=aarch64-linux-android- KCFLAGS=-mno-android -j8
	out/host/linux-x86/bin/dtbTool -o out/target/product/<target>/dt.img -s 2048 -p out/target/product/<target>/obj/KERNEL_OBJ/scripts/dtc/ out/target/product/<target>/obj/KERNEL_OBJ/arch/arm64/boot/dts/
	rm out/target/product/<target>/kernel
	cp out/target/product/<target>/obj/KERNEL_OBJ/arch/arm64/boot/Image.gz-dtb out/target/product/<target>/kernel
	rm out/target/product/<target>/boot.img
	make bootimage-nodeps

How to set SUID for a thread.

    struct cred *p = (struct cred*)current->real_cred;
    kuid_t root;
    root.val = 0;
    p->euid = root;


## Summary

## Reference
- Linux进程的实际用户ID和有效用户ID
- Android adb setuid提权漏洞的分析
- http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-2942 
- http://security.stackexchange.com/questions/68442/escalating-from-apache-shell-to-root/68482#68482
- https://www.exploit-db.com/exploits/15285/
- http://www.cis.syr.edu/~wedu/seed/Labs_12.04/Vulnerability/Buffer_Overflow/
- http://insecure.org/stf/smashstack.html