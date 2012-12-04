---
layout: master
title: Machine_desc callings
---

## Overview

	MACHINE_START(xxxxx)
        .map_io = xxx,
        .reserve = xxx,
        .init_irq = xxx,
        .handle_irq = xxx,
        .timer = xxx,
        .init_machine = xxx,
        .init_early = xxx,
        .init_very_early = xxx,
        .restart = xxx,
	MACHINE_END


## Calling sequence

MACHINE_START

Defined in :
	
	arch/arm/include/asm/mach/arch.h

## Calling sequence

### init_machine


	start_kernel()--->setup_arch()
			  --->rest_init--->kernel_init(){kernel_thread}
								--->do_basic_setup--->
									--->do_initcalls()
										--->customize_machine(){__initcall3_start}--->.init_machine


- start_kernel()

	/init/main.c

- setup_arch()

/arch/arm/kernel/setup.c
	

	void __init setup_arch(char **cmdline_p)
	{
		struct machine_desc *mdesc;

		setup_processor();
		mdesc = setup_machine_fdt(__atags_pointer);
		if (!mdesc)
			mdesc = setup_machine_tags(machine_arch_type);
		machine_desc = mdesc;
		machine_name = mdesc->name;


- setup\_machine_fdt

>  * setup\_machine_fdt - Machine setup when an dtb was passed to the kernel
>  * @dt_phys: physical address of dt blob
>  *
>  * If a dtb was passed to the kernel in r2, then use it to choose the
>  * correct machine_desc and to setup the system.
>  */

- customize_machine

		static int __init customize_machine(void)
		{
			/* customizes platform devices, or adds new ones */
			if (machine_desc->init_machine)
				machine_desc->init_machine();
			return 0;
		}

		arch\_initcall(customize_machine);

		#define arch_initcall(fn)               __define_initcall("3",fn,3)

			#define __define_initcall(level,fn) \
			static initcall_t __initcall_##fn __used \
			__attribute__((__section__(".initcall" level ".init"))) = fn

		
		/* initcalls are now grouped by functionality into separate 
		 * subsections. Ordering inside the subsections is determined
		 * by link order. 
		 * For backwards compatibility, initcall() puts the call in 
		 * the device init subsection.
		 *
		 * The `id' arg to __define_initcall() is needed so that multiple initcalls
		 * can point at the same handler without causing duplicate-symbol build errors.
		 */



- do_initcalls


		Main.c (init)	22012	11/26/2012

		static void __init do_initcalls(void)
		{
			int level;
		
			for (level = 0; level < ARRAY_SIZE(initcall_levels) - 1; level++)
				do_initcall_level(level);
		}


### init_early

start_kernel()--->setup_arch()--->init_very_early()

called in 

		/arch/arm/kernel/setup.c



		void __init setup_arch(char **cmdline_p)
		{
			struct machine_desc *mdesc;
		
			setup_processor();
			mdesc = setup_machine_fdt(__atags_pointer);
			if (!mdesc)
				mdesc = setup_machine_tags(machine_arch_type);
			machine_desc = mdesc;
			machine_name = mdesc->name;
		
			setup_dma_zone(mdesc);
		
			if (mdesc->restart_mode)
				reboot_setup(&mdesc->restart_mode);
		
			if (mdesc->init_very_early)
				mdesc->init_very_early();

			sanity_check_meminfo();
			arm_memblock_init(&meminfo, mdesc);
		
			paging_init(mdesc);
			request_standard_resources(mdesc);
		
			if (mdesc->restart)
				arm_pm_restart = mdesc->restart;

		
		#ifdef CONFIG_MULTI_IRQ_HANDLER
			handle_arch_irq = mdesc->handle_irq;
		#endif
		
		#ifdef CONFIG_VT
		#if defined(CONFIG_VGA_CONSOLE)
			conswitchp = &vga_con;
		#elif defined(CONFIG_DUMMY_CONSOLE)
			conswitchp = &dummy_con;
		#endif
		#endif
		
			if (mdesc->init_early)
				mdesc->init_early();
		}
		


### init\_very_early


start_kernel()--->setup_arch()--->init_very_early()--->init_early()->

##__init

### definition

	#define __init		__section(.init.text) __cold notrace
	#define __initdata	__section(.init.data)
	#define __initconst	__section(.init.rodata)
	#define __exitdata	__section(.exit.data)
	#define __exit_call	__used __section(.exitcall.exit)

These macros are used to mark some functions or 
initialized data (doesn't apply to uninitialized data)
as `initialization' functions. 

The kernel can take this as hint that the function is used only during the initialization
phase and free up used memory resources after.



### Usage

#### For functions:

You should add **__init** immediately before the function name, like:

	 *
	 * static void __init initme(int x, int y)
	 * {
	 *    extern int z; z = x * y;
	 * }
	 *

If the function has a prototype somewhere, you can also add
__init between closing brace of the prototype and semicolon:

	 * extern int initialize_foobar_device(int, int, int) __init;


#### For initialized data:

You should insert **__initdata** between the variable name and equal
sign followed by value, e.g.:

	 *
	 * static int init_variable __initdata = 0;
	 * static const char linux_logo[] __initconst = { 0x32, 0x36, ... };
	 *

 * Don't forget to initialize data not at file scope, i.e. within a function,
 * as gcc otherwise puts the data into the bss section and not into the init
 * section.
 * 
 * Also note, that this data cannot be "const".
## Reference

