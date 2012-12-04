---
layout: master
title: Linux Device Driver
---

## Overview


## Linux Device Model 

a set of data structures


Parts

- Power management
- startup and shutdown
- Communications with user space
- System configuration

### Mode

A split implementation model:“Split” Devices and Drivers

Analogous to C++ class “methods” and “data”

	• “Driver” is separated from “device”
	• Promotes reuse, portability
	• See struct device_driver and struct device

## Basic

### module_init  

module_init宏在MODULE宏有没有定义的情况下展开的内容是不同的，如果这个宏没有定义，即模块是要编译进内核的(obj-y)。

#### build-in

	#define module_init(x) __initcall(x);
	#define __initcall(fn) device_initcall(fn)
	#define __define_initcall(level,fn,id) \
		static initcall_t __initcall_##fn##id __used \
		__attribute__((__section__(".initcall" level ".init"))) = fn


	int gpio_init(void)，那么module_init(gpio_init)实际上等于:

	static initcall_t  __initcall_gpio_init_6 __used __attribute__((__section__(".initcall6.init"))) = gpio_init;

就是声明一类型为initcall\_t（typedef int (*initcall\_t)(void)）函数指针类型的变量\_\_initcall\_gpio\_init\_6并将gpio_init赋值与它。

存放在： 
".initcall6.init"


vmlinux.lds

	.initcall.init : AT(ADDR(.initcall.init) - (0xc0000000 -0x00000000)) {
	   __initcall_start = .;
	   *(.initcallearly.init) __early_initcall_end = .; *(.initcall0.init) *(.initcall0s.init) *(.initcall1.init) *(.initcall1s.init) *(.initcall2.init) *(.initcall2s.init) *(.initcall3.init) *(.initcall3s.init) *(.initcall4.init) *(.initcall4s.init) *(.initcall5.init) *(.initcall5s.init) *(.initcallrootfs.init) *(.initcall6.init) *(.initcall6s.init) *(.initcall7.init) *(.initcall7s.init)
	   __initcall_end = .;
   }

调用：

do_initcalls

	start、_kernel()->rest\_init()->kernel_init()->do_basic_setup()->do_initcalls()。


module_exit在静态编译的时候没有意义,因为静态编译的驱动无法卸载!

#### module


	#define module_init(initfn) \
	static inline initcall_t __inittest(void) \
	{ return initfn; } \
	int init_module(void) __attribute__((alias(#initfn)));


#### 1.验证加载函数的格式

	static inline initcall_t __inittest(void)	 \
	{ return initfn; }

这个函数的作用是验证我们穿过来的加载函数格式是否正确，linux内核规定加载函数的的原型是：

	typedef int (*initcall_t)(void);

所以我们写加载函数的时候必须是返回值为int参数为void的函数，这个在内核里要求比较严格，所以我们写加载函数的时候必须按照这个约定。

#### 2.定义别名

	int init_module(void) __attribute__((alias(#initfn)));

这段代码的作用是给我们的加载函数定义一个别名，别名就是我们前面提到的init_module，这样insmod就能够执行我们的加载函数了。

module\_exit的作用和module_init一样，同样也是验证函数格式和定义别名。

### insmod
 

- 1.get_kernel_info函数负责取得kernel中先以注册的modules，放入module_stat中. 
- 2.set_ncv_prefix(NULL);判断symbolname中是否有前缀，象_smp之类，一般没有。 
- 3.检查是否已有同名的module。 
- 4.obj_load，将。o文件读入到struct obj_file结构f中。 
- 5.比较kernel版本和module的版本，在版本的判断中，不是想象中的那样简单，还有是否有checksum的逻 辑关系。 
- 6.add_kernel_symbols替换.o中的symbol为ksyms中的符号值 
- 7.create_this_module(f, m_name)生成module结构，加入module_list中。 
- 8.obj_check_undefineds检查是否还有un_def的symbol 
- 9.add_archdata添加结构相关的section，不过i386没什么用。 
- 10.add_kallsyms如果symbol使用的都是kernel提供的，就添加一个.kallsyms section 
- 11.obj_load_size计算module的大小 
- 12.create_module调用sys_create_module系统调用创建模块，分配module的空间 
- 13.obj_relocate重定位module文件中.text中的地址 
- 14.init_module先在用户空间创建module结构的image影响，是由sys_init_module系统调用实现向kernel的 copy。


## struct device

Represents instances of “devices”

### what is device

- Physical hardware component e.g. chip
- Sub-functions of a complex chip
- Instance of an abstract component e.g. “LED”, “GPIO”


- Linux devices DON’T have interfaces(open/close)!
- Linux devices are inanimate data objects
### struct

	struct device {
		...
		const char *init_name;
		struct device *parent;
		struct kobject kobj;
		struct bus_type *bus;
		struct device_driver *driver;
		void *platform_data;
		...
	};

### register

#### when

- Board startup, usually
- Can be done anytime, really

Way
	
	p = platform_device_alloc("foo", -1);
	ret = platform_device_add(p);

## struct device_driver

Animates instances of struct device objects

### struct

		struct device_driver {
			int (*probe ) (struct device *dev);
			int (*remove ) (struct device *dev);
			void (*shutdown) (struct device *dev);
			int (*suspend ) (struct device *dev,
			pm_message_t state);
			int (*resume ) (struct device *dev);
			...
			...
			const char *name;
			const struct dev_pm_ops *pm;
			struct module *owner;
			struct driver_private *p;
			const struct attribute_group **groups;
			...
		};


### probe()

Invoked when a device, driver .name match occurs

	...
	ret = device_add(struct device *dev);
	...
	ret = driver_register(struct device_driver *driver);

### register

#### When

-	Module initialization, usually
-	During do_initcalls() otherwise

	module\_init(foo_init);

## Device Attributes

Similar to device nodes, but:

- Tightly coupled to device, model
- Limited to one page of data
- Only “show” and “store” methods
- No streaming capability

### “Show” Method

### “Store” Method

### DEVICE_ATTR


## Reference

