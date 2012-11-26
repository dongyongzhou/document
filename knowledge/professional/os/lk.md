---
layout: master
title: LK
---

## Overview

LK 是 Little Kernel

它是 appsbl（Applications ARM Boot Loader）流程代码，

little kernel 是小内核小操作系统。


## Code structure

code directory

	bootable/bootloadler/lk

folders

           +app            // 应用相关
           +arch           // arm 体系
           +dev            // 设备相关
           +include      // 头文件
           +kernel        // lk系统相关  
           +platform    // 相关驱动
           +projiect     // makefile文件
           +scripts      // Jtag 脚本
           +target        // 具体板子相关



## building

## bootup sequence




### ENTRY（_start）指定 LK 从_start 函数开始

	bootable/bootloadler/lk/arch/arm/ssystem-onesegment.ld 


### _start 

   lk/arch/arm/crt0.S中 

	reset->Lstack_setup->kmain


   crt0.S 主要做一些基本的 CPU 的初始化再通过 bl  kmain ；跳转到 C 代码中。
          
### kmain

	lk/kernel/main.c 中

#### kmain works

- 1、本身 lk 这个系统模块的初始化
- 2、boot 的启动初始化动作。


#### analyze

	// get us into some sort of thread context
	thread_init_early();

	// early arch stuff
	arch_early_init();

	// do any super early platform initialization
	platform_early_init();

	// do any super early target initialization
	target_early_init();

	dprintf(INFO, "welcome to lk\n\n");
	
	// deal with any static constructors
	dprintf(SPEW, "calling constructors\n");
	call_constructors();

	// bring up the kernel heap
	dprintf(SPEW, "initializing heap\n");
	heap_init();

	// initialize the threading system
	dprintf(SPEW, "initializing threads\n");
	thread_init();

	// initialize the dpc system
	dprintf(SPEW, "initializing dpc\n");
	dpc_init();

	// initialize kernel timers
	dprintf(SPEW, "initializing timers\n");
	timer_init();

		#if (!ENABLE_NANDWRITE)
	// create a thread to complete system initialization
	dprintf(SPEW, "creating bootstrap completion thread\n");
	thread_resume(thread_create("bootstrap2", &bootstrap2, NULL, DEFAULT_PRIORITY, DEFAULT_STACK_SIZE));

	// enable interrupts
	exit_critical_section();

	// become the idle thread
	thread_become_idle();
		#else
        bootstrap_nandwrite();
		#endif


thread_resume(thread_create("bootstrap2", &bootstrap2, NULL, DEFAULT_PRIORITY, DEFAULT_STACK_SIZE));

### bootstrap2

	arch_init();

	// initialize the rest of the platform
	dprintf(SPEW, "initializing platform\n");
	platform_init();
	dprintf(SPEW, "calling apps_init()\n");
	apps_init();

#### apps_init


	/* call all the init routines */
	for (app = &__apps_start; app != &__apps_end; app++) {
		if (app->init)
			app->init(app);
	}

Aboot.c (app\aboot)	42379	11/23/2012

	APP_START(aboot)
		.init = aboot_init,


### aboot_init

#### key checking and enter recovery||fastboot || normal boot

	/* Check if we should do something other than booting up */
	if (keys_get_state(KEY_HOME) != 0)
		boot_into_recovery = 1;
	if (keys_get_state(KEY_VOLUMEUP) != 0)
		boot_into_recovery = 1;
	if(!boot_into_recovery)
	{
		if (keys_get_state(KEY_BACK) != 0)
			goto fastboot;
		if (keys_get_state(KEY_VOLUMEDOWN) != 0)
			goto fastboot;
	}

#### boot_linux from XXX


	if (target_is_emmc_boot())
	{
 		...
		boot_linux_from_mmc();
	}
	else
	{
		recovery_init();
		...
		boot_linux_from_flash();
	}


#### boot\_linux\_from_mmc

	if (!boot_into_recovery) {
		index = partition_get_index("boot");
		ptn = partition_get_offset(index);
		if(ptn == 0) {
			dprintf(CRITICAL, "ERROR: No boot partition found\n");
                    return -1;
		}
	}
	else {
		index = partition_get_index("recovery");
		ptn = partition_get_offset(index);
		if(ptn == 0) {
			dprintf(CRITICAL, "ERROR: No recovery partition found\n");
                    return -1;
		}
	}

		image_addr = (unsigned char *)target_get_scratch_address();

		/* Move kernel, ramdisk and device tree to correct address */
		memmove((void*) hdr->kernel_addr, (char *)(image_addr + page_size), hdr->kernel_size);
		memmove((void*) hdr->ramdisk_addr, (char *)(image_addr + page_size + kernel_actual), hdr->ramdisk_size);



	boot_linux((void *)hdr->kernel_addr, (unsigned *) hdr->tags_addr,
		   (const char *)cmdline, board_machtype(),
		   (void *)hdr->ramdisk_addr, hdr->ramdisk_size);

#### boot\_linux

	void (*entry)(unsigned, unsigned, unsigned*) = kernel;
	entry(0, machtype, tags);


machine type is stored in R1?

## Reference

