---
layout: master
title: mach-types
---

##1 Overview

mach-types

##2 Analyze

###2.1 files 

	kernel/arch/arm/tools/gen-mach-types
	kernel/arch/arm/tools/mach-types
	kernel/arch/arm/tools/Makefile


include/generated/mach-types.h

include/generated/mach-types.h


generated in building stage.


###2.2 Makefile

		include/generated/mach-types.h: $(src)/gen-mach-types $(src)/mach-types
        @echo '  Generating $@'
        @mkdir -p $(dir $@)
        $(Q)$(AWK) -f $^ > $@ || { rm -f $@; /bin/false; }


@awk -f gen-mach-types mach-types >include/generated/mach-types.h


AWK是一种优良的文本处理工具

使用-f选项调用awk程序。awk允许将一段awk程序写入一个文本文件，然后在awk命令行中用-f选项调用并执行这段程序

###2.3 gen-mach-types

awk程序

####2.3.1 basic 

在awk 中两个特别的表达式，BEGIN和END，这两者都可用于pattern中，
提供BEGIN和END的作用是给程序赋予初始状态和在程序结束之后执行一些扫尾的工作。

- 任何在BEGIN之后列出的操作（在{}内）将在awk开始扫描输入之前执行，
- 而END之后列出的操作将在扫描完全部的输入之后执行。

因此，通常使用BEGIN来显示变量和预置（初始化）变量，使用END来输出最终结果。

####2.3.2 analyze

	5 BEGIN   { nr = 0 }

给变量nr赋初值为0。

  	6 /^#/    { next }
  	7 /^[     ]*$/ { next }

如果遇到目标文件中以 #开头的行及空白行或者空行
，则跳过这一行。


	9 NF == 4 {
 	10           machine_is[nr] = "machine_is_"$1;
 	11           config[nr] = "CONFIG_"$2;
 	12           mach_type[nr] = "MACH_TYPE_"$3;
 	13           num[nr] = $4; nr++
 	14         }

 如果此行的字段数为 4 ，则将此行的每一段添加上对应的头后保存到指定的数组中，$1,$2,$3,$4分别为四个字段的数据。语句语句执行完后，mach-types文件中的所有字段就被保存下来了


 24 END     {

	 39               printf("#define %-30s %d\n", mach_type[i], num[i]);

定义对应的芯片类型为对应的机器号。


	  43           for (i = 0; i < nr; i++)
 	  44             if (num[i] ~ /..*/) {
 	  45               printf("#ifdef %s\n", config[i]);
 	  46               printf("# ifdef machine_arch_type\n");
	  47               printf("#  undef machine_arch_type\n");
	  48               printf("#  define machine_arch_type\t__machine_arch_type\n");
 	  49               printf("# else\n");
	  50               printf("#  define machine_arch_type\t%s\n", mach_type[i]);
	  51               printf("# endif\n");
 	  52               printf("# define %s()\t(machine_arch_type == %s)\n", machine_is[i], mach_type[i]);
	  53               printf("#else\n");
	  54               printf("# define %s()\t(0)\n", machine_is[i]);
 	  55               printf("#endif\n\n");
 	  56             }


写入每个芯片对应的配置宏定义

	

##3 bootup sequence

###3.1 aboot

####passing machine type to Linux kernel

## Reference

