---
layout: master
title: Linux sys file system 
---

##1 sysfs Overview

sysfs 文件系统按约定总是被挂载在 /sys上

Linux用户态访问内核数据方法有多种，除了基本的驱动设备节点外，还有虚拟的基于内存的文件系统procfs和sysfs。

查看和设定内核参数，更适合使用虚拟文件系统procfs和sysfs。

###1.1 procfs与sysfs比较

一个 proc 虚拟文件可能有内部格式，并且读写格式不一样，代表不同的操作，应用程序中读到了这个文件的内容一般还需要进行字符串解析，而在写入时需要先用字符串>格式化按指定的格式写入字符串进行操作；

sysfs 的设计原则是一个属性文件只做一件事情， sysfs 属性文件一般只有一个值，直接读取或写入。

新设计的内核机制应该尽量使用 sysfs 机制，而将 proc 保留给纯净的“进程文件系统”。

##2 /sys 目录结构
 
+ /sys/devices 内核对系统中**所有设备的分层次表达模型**，也是 /sys 文件系统管理设备的**最重要**的目录结构
+ /sys/dev	维护一个**按字符设备和块设备的主次号码**(major:minor)链接到真实的设备(/sys/devices下)的符号链接文件，它是在内核 2.6.26 首次引入；
+ /sys/bus	内核设备**按总线类型**分层放置的目录结构， devices 中的所有设备都是连接于某种总线之下，在这里的每一种具体总线之下可以找到每一个具体设备的符号链接，它也是构成 Linux 统一设备模型的一部分；
+ /sys/class	**按设备功能分类**的设备模型，如系统所有输入设备都会出现在 /sys/class/input 之下，而不论它们是以何种总线连接到系统。它也是构成 Linux 统一设备模型的一部分；
+ /sys/block	系统中当前所有的块设备所在，按照功能来说放置在 /sys/class 之下会更合适，但只是由于历史遗留因素而一直存在于 /sys/block, 但从 2.6.22 开始就已标记为过时，只有在打开了 CONFIG_SYSFS_DEPRECATED 配置下编译才会有这个目录的存在，并且在 2.6.26 内核中已正式移到 /sys/class/block, 旧的接口 /sys/block 为了向后兼容保留存在，但其中的内容已经变为指向它们在 /sys/devices/ 中真实设备的符号链接文件；
+ /sys/firmware	系统加载固件机制的对用户空间的接口，关于固件有专用于固件加载的一套API，在 LDD3 一书中有关于内核支持固件加载机制的更详细的介绍；
+ /sys/fs	用于描述系统中所有文件系统，包括文件系统本身和按文件系统分类存放的已挂载点，但目前只有 fuse,gfs2 等少数文件系统支持 sysfs 接口，一些传统的虚拟文件系统(VFS)层次控制参数仍然在 sysctl (/proc/sys/fs) 接口中中；
+ /sys/kernel	**内核所有可调整参数**的位置，目前只有 uevent_helper, kexec_loaded, mm, 和新式的 slab 分配器等几项较新的设计在使用它，其它内核可调整参数仍然位于 sysctl (/proc/sys/kernel) 接口中 ;
+ /sys/module	系统中**所有模块的信息**，不论这些模块是以内联(inlined)方式编译到内核映像文件(vmlinuz)中还是编译为外部模块(ko文件)，都可能会出现在 /sys/module 中：

    - 编译为**外部模块**(ko文件)在加载后会出现对应的 /sys/module/<module_name>/, 并且在这个目录下会出现一些属性文件和属性目录来表示此外部模块的一些信息，如版本号、加载状态、所提供的驱动程序等；
    - 编译为**内联方式**的模块则只在当它有非0属性的模块参数时会出现对应的 /sys/module/<module_name>, 这些模块的可用参数会出现在 /sys/modules/<modname>/parameters/<param_name> 中，
    如 /sys/module/printk/parameters/time 这个可读写参数控制着内联模块 printk 在打印内核消息时是否加上时间前缀；
    - 所有内联模块的参数也可以由 "<module_name>.<param_name>=<value>" 的形式写在内核启动参数上，如启动内核时加上参数 "printk.time=1" 与 向 "/sys/module/printk/parameters/time" 写入1的效果相同；
    没有非0属性参数的内联模块不会出现于此。

+ /sys/power	系统中电源选项，这个目录下有几个属性文件可以用于控制整个机器的电源状态，如可以向其中写入控制命令让机器关机、重启等。
+ /sys/slab (对应 2.6.23 内核，在 2.6.24 以后移至 /sys/kernel/slab)	从2.6.23 开始可以选择 SLAB 内存分配器的实现，并且新的 SLUB（Unqueued Slab Allocator）被设置为缺省值；如果编译了此选项，在 /sys 下就会出现 /sys/slab ，里面有每一个 kmem_cache 结构体的可调整参数。对应于旧的 SLAB 内存分配器下的 /proc/slabinfo 动态调整接口，新式的 /sys/kernel/slab/<slab_name> 接口中的各项信息和可调整项显得更为清晰。


###2.1 /sys/devices

/sys/devices/ 目录下是按照设备的基本总线类型分类的目录

##3 sysfs 与设备驱动模型

Linux对设备层次进行分层

总线1->总线2->设备

sysfs 将这些有层次结构的设备以用户程序可见的方式表达出来，利用文件系统的目录树结构。在这个模型中，有几种基本类型，它们的对应关系如下所示


<table id="devices" class="red-table">
          <thead>
                <th>类型</th>
                <th>包含的内容</th>
                <th>对应内核数据结构</th>
                <th>对应/sys项</th>

          </thead>

          <tr>
                <td>设备(Devices)</td>
                <td>设备是此模型中最基本的类型，以设备本身的连接按层次组织</td>
                <td>struct device</td>
                <td>/sys/devices/*/*/.../</td>
          </tr>
          <tr>
                <td>设备驱动(Device Drivers)</td>
                <td>在一个系统中安装多个相同设备，只需要一份驱动程序的支持</td>
                <td>struct device_driver</td>
                <td>/sys/bus/pci/drivers/*/</td>
          </tr>
          <tr>
                <td>总线类型(Bus Types)</td>
                <td>在整个总线级别对此总线上连接的所有设备进行管理</td>
                <td>struct bus_type</td>
                <td>/sys/bus/*/</td>
          </tr>
          <tr>
                <td>设备类别(Device Classes)</td>
                <td>这是按照功能进行分类组织的设备层次树；如 USB 接口和 PS/2 接口的鼠标都是输入设备，都会出现在 /sys/class/input/ 下</td>
                <td>struct class</td>
                <td>/sys/class/*/</td>
          </tr>
</table>


##4 深入理解sysfs


Linux 统一设备模型又是以kobject和kset两种基本数据结构进行树型和链表型结构组织的：

###4.1 kobject

在 Linux 设备模型中最基本的对象，它的功能是提供引用计数和维持父子(parent)结构、平级(sibling)目录关系，device, device_driver 等各对象都是以 kobject 基础功能之上实现的；

    struct kobject {
        const char              *name;
        struct list_head        entry;
        struct kobject          *parent;
        struct kset             *kset;
        struct kobj_type        *ktype;
        struct sysfs_dirent     *sd;
        struct kref             kref;
        unsigned int state_initialized:1;
	    unsigned int state_in_sysfs:1;
        unsigned int state_add_uevent_sent:1;
        unsigned int state_remove_uevent_sent:1;
    }; 

其中 struct kref 内含一个 atomic_t 类型用于引用计数， parent 是单个指向父节点的指针， entry 用于父 kset 以链表头结构将 kobject 结构维护成双向链表；

其中sysfs_dirent 将直接对应一个属性文件

###4.2 kset

它用来对同类型对象提供一个包装集合，在内核数据结构上它也是由内嵌一个 kboject 实现，因而它同时也是一个 kobject (面向对象 OOP 概念中的继承关系) ，具有 kobject 的全部功能；

    struct kset {
        struct list_head list;
        spinlock_t list_lock;
        struct kobject kobj;
        struct kset_uevent_ops *uevent_ops;
    }; 

其中的 struct list_head list 用于将集合中的 kobject 按 struct list_head entry 维护成双向链表；



###4.3 sysfs文件系统实现

sysfs 是一种基于 ramfs 实现的内存文件系统，与其它同样以 ramfs 实现的内存文件系统(configfs,debugfs,tmpfs,...)类似， sysfs 也是直接以 VFS 中的 struct inode 和 struct dentry 等 VFS 层次的结构体直接实现文件系统中的各种对象；同时在每个文件系统的私有数据 (如 dentry->d_fsdata 等位置) 上，使用了称为 struct sysfs_dirent 的结构用于表示 /sys 中的每一个目录项。

    struct sysfs_dirent {
        atomic_t                s_count;
        atomic_t                s_active;
        struct sysfs_dirent     *s_parent;
        struct sysfs_dirent     *s_sibling;
        const char              *s_name;

        union {
                struct sysfs_elem_dir           s_dir;
                struct sysfs_elem_symlink       s_symlink;
                struct sysfs_elem_attr          s_attr;
                struct sysfs_elem_bin_attr      s_bin_attr;
        };

        unsigned int            s_flags;
        ino_t                   s_ino;
        umode_t                 s_mode;
        struct iattr            *s_iattr;
}; 

kobject 对象有 sysfs_dirent 的指针，因此在sysfs中是用同一种 struct sysfs_dirent 来统一设备模型中的 kset/kobject/attr/attr_group.

具体在数据结构成员上， sysfs_dirent 上有一个 union 共用体包含四种不同的结构，分别是目录、符号链接文件、属性文件、二进制属性文件；其中目录类型可以对应 kobject，在相应的 s_dir 中也有对 kobject 的指针，因此在内核数据结构， kobject 与 sysfs_dirent 是互相引用的；


### 4.4 /sys 目录结构技术描述

- 在 /sys 根目录之下的都是 kset，它们组织了 /sys 的顶层目录视图；
- 在部分 kset 下有二级或更深层次的 kset；
- 每个 kset 目录下再包含着一个或多个 kobject，这表示一个集合所包含的 kobject 结构体；
- 在 kobject 下有属性(attrs)文件和属性组(attr_group)，属性组就是组织属性的一个目录，它们一起向用户层提供了表示和操作这个 kobject 的属性特征的接口；
- 在 kobject 下还有一些符号链接文件，指向其它的 kobject，这些符号链接文件用于组织上面所说的 device, driver, bus_type, class, module 之间的关系；
- 不同类型如设备类型的、设备驱动类型的 kobject 都有不同的属性，不同驱动程序支持的 sysfs 接口也有不同的属性文件；而相同类型的设备上有很多相同的属性文件；

注意，此内容是按照最新开发中的 2.6.28 内核的更新组织的，

LDD3 等位置中有提到 sysfs 中曾有一种管理对象称为 subsys (子系统对象)，在最新的内核中经过重构认为它是不需要的，它的功能完全可以由 kset 代替，也就是说 sysfs 中只需要一种管理结构是 kset，一种代表具体对象的结构是 kobject，在 kobject 下再用属性文件表示这个对象所具有的属性；

##5  sysfs 属性用法

###5.1 uevent

在 sysfs 下的很多 kobject 下都有 uevent 属性

它主要用于内核与 udev (自动设备发现程序)之间的一个通信接口；从 udev 本身与内核的通信接口 netlink 协议套接字来说，它并不需要知道设备的 uevent 属性文件，但多了 uevent 这样一个接口，可用于 udevmonitor 通过内核向 udevd (udev 后台程序)发送消息，也可用于检查设备本身所支持的 netlink 消息上的环境变量，这个特性一般用于开发人员调试 udev 规则文件， udevtrigger 这个调试工具本身就是以写各设备的 uevent 属性文件实现的。

这些 /devices/ 属性文件都支持写入，当前支持写入的参数有 "add","remove","change","move","online","offline"。如，写入 "add"，这样可以向 udevd 发送一条 netlink 消息，让它再重新一遍相关的 udev 规则文件；这个功能对开发人员调试 udev 规则文件很有用。


### 5.2 查找设备和驱动对应关系

模块是内核驱动编程的最佳选择，而一个模块有可能提供多个驱动程序，因而在未知一个设备在用哪一个驱动的情况下可以:

a 先从 /sys/module/ 查找相应模块的情况，再从 drivers/ 发现出真正的驱动程序。

b 或者也可以完全反过来利用这些信息，先用 lspci/lshw 等工具找到 /sys/devices/ 下的设备节点，再从其设备的 driver 链接找到 /sys/bus/*/drivers/ 下的 device_driver, 再从 device_driver 下的 module 链接找到 /sys/module/*/，这样就可以得到已加载模块中空间是哪一个模块在给一个设备提供驱动程序。

##6 sysfs代码分析

###6.1 sysfs 属性定义

在内核中， sysfs 属性一般是由 __ATTR 系列的宏来声明的，如对设备的使用 DEVICE_ATTR ，对总线使用 BUS_ATTR ，对驱动使用 DRIVER_ATTR ，对类别(class)使用 CLASS_ATTR, 这四个高级的宏来自于 <include/linux/device.h>, 都是以更低层的来自 <include/linux/sysfs.h> 中的 __ATTR/__ATRR_RO 宏实现；

<include/linux/device.h> 头文件提供的这四个宏，分别应用于总线/类别/驱动/设备四种内核数据结构对象上：

    #define BUS_ATTR(_name, _mode, _show, _store)   \
    struct bus_attribute bus_attr_##_name = __ATTR(_name, _mode, _show, _store)
    
    #define CLASS_ATTR(_name, _mode, _show, _store)                 \
    struct class_attribute class_attr_##_name = __ATTR(_name, _mode, _show, _store)
    
    #define DRIVER_ATTR(_name, _mode, _show, _store)        \
    struct driver_attribute driver_attr_##_name =           \
            __ATTR(_name, _mode, _show, _store)

    #define DEVICE_ATTR(_name, _mode, _show, _store) \
    struct device_attribute dev_attr_##_name = __ATTR(_name, _mode, _show, _store)


上面四种都是 Linux 统一设备模型所添加的高级接口,上面的总线/类别/驱动/设备四个接口都是以这里的__ATTR实现的，如果使用 sysfs 所提供的底层接口的话，则可以直接使用下面两个，定义来自 <include/linux/sysfs.h> ：

    #define __ATTR(_name,_mode,_show,_store) { \
        .attr = {.name = __stringify(_name), .mode = _mode },   \
        .show   = _show,                                        \
	    .store  = _store,                                       \
    }
    
    #define __ATTR_RO(_name) { \
        .attr   = { .name = __stringify(_name), .mode = 0444 }, \
        .show   = _name##_show,                                 \
    }

宏声明有四个参数，分别是名称、权限位、读函数、写函数。

因此读写功能与权限位是对应的，因为 DEVICE_ATTR 把权限位声明与真正的读写是否实现放在了一起，减少了出现不一致的可能。

###6.2 sysfs 属性权限

/linux/stat.h

    #define S_IRWXU 00700
    #define S_IRUSR 00400
    #define S_IWUSR 00200
    #define S_IXUSR 00100
    
    #define S_IRWXG 00070
    #define S_IRGRP 00040
    #define S_IWGRP 00020
    #define S_IXGRP 00010
    
    #define S_IRWXO 00007
    #define S_IROTH 00004
    #define S_IWOTH 00002
    #define S_IXOTH 00001

    #define S_IRWXUGO       (S_IRWXU|S_IRWXG|S_IRWXO)
    #define S_IALLUGO       (S_ISUID|S_ISGID|S_ISVTX|S_IRWXUGO)
    #define S_IRUGO         (S_IRUSR|S_IRGRP|S_IROTH)
    #define S_IWUGO         (S_IWUSR|S_IWGRP|S_IWOTH)
    #define S_IXUGO         (S_IXUSR|S_IXGRP|S_IXOTH)

###6.1 sysfs 属性读写函数

如device_attribute

    /* interface for exporting device attributes */
    struct device_attribute {
        struct attribute        attr;
        ssize_t (*show)(struct device *dev, struct device_attribute *attr,
                        char *buf);
        ssize_t (*store)(struct device *dev, struct device_attribute *attr,
                         const char *buf, size_t count);
    };

从头文件中还可以找到 show/store 函数的原型，注意到它和虚拟字符设备或 proc 项的 read/write 的作用很类似，但有一点不同是 show/store 函数上的 **buf/count 参数是在 sysfs 层已作了用户区/内核区的内存复制**，虚拟字符设备上常见的 __user 属性在这里并不需要，因而也不需要多一次 copy_from_user/copy_to_user, 在 show/store 函数参数上的 buf/count 参数已经是内核区的地址，可以直接操作。

##7 sysfs编程

设备驱动程序中需要与用户层的接口, 最好的方法还是使用 sysfs 属性支持，一切在用户层是可见的透明，且增加的代码量是最少的，可维护性也最好

例子如device

###7.1 定义sysfs属性文件

头文件

    linux/device.h
    linux/sysfs.h

定义属性文件

    static DEVICE_ATTR(yyy, S_IRUGO | S_IWUSR, xxxx_show_yyy, xxxx_store_yyy);

###7.2 实现属性文件操作接口

    ssize_t xxxx_show_yyy(struct device *dev,
                struct device_attribute *attr, char *buf)
    {
        return snprintf(buf, PAGE_SIZE, "%d\n", zzz);
    }
    
    ssize_t xxxx_store_yyy(struct device *dev,
                 struct device_attribute *attr, const char *buf, size_t count)
    {
        //store operation
        int tmp;

        sscanf(buf, "%d", &tmp);//接受数字输入，并转化为int格式数据
        ....
    }

###7.3 属性文件注册和撤销

属性文件注册

place: init or .probe

    ret = device_create_file(xxxx_device, &dev_attr_yyy);
        if (ret) {
                pr_err("%s: device_create_file(%s)=%d\n",
                                __func__, dev_attr_yyy.attr.name, ret);
                goto device_create_file_fail;
        }

属性文件撤销

place: exit or .remove

    device_remove_file(xxxx_device, &dev_attr_yyy);


## Reference

kernel/Documentation/sysfs-rules.txt

http://www.ibm.com/developerworks/cn/linux/l-cn-sysfs/
