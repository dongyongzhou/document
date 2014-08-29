---
layout: master
title:  compilation
---

# Basic

##映像文件

arm映像文件其实就是可执行文件，包括bin或hex两种格式，可以直接烧到rom里执行。

映像文件一般由域组成，域最多由三个**输出段**组成(RO,RW,ZI)组成，输出段又由**输入段**组成。

###加载域、运行域

所谓**域**，指的就是整个bin映像文件所处在的区域，它又分为加载域和运行域。对于嵌入式系统而言，程序映象都是存储在Flash存储器等一些非易失性器件中的，而在运行时，程序中的RW段必须重新装载到可读写的RAM中。这就涉及到程序的加载时域和运行时域。

- **加载域**就是映像文件被静态存放的工作区域，一般来说flash里的整个bin文件所在的地址空间就是加载域，
- 当然在程序一般都不会放在flash里执行，一般都会搬到sdram里运行工作，它们在被搬到sdram里工作所处的地址空间就是**运行域**。

简单来说，程序的加载时域就是指程序烧入Flash中的状态，运行时域是指程序执行时的状态。

###输入段、输出段
- 我们输入的代码，一般有代码部分和数据部分，这就是所谓的**输入段**
- 经过编译后就变成了bin文件中ro段和rw段，还有所谓的zi段，这就是**输出段**。

对于加载域中的输出段，一般来说ro段后面紧跟着rw段，rw段后面紧跟着zi段。
在运行域中这些输出段并不连续，但rw和zi一定是连着的。zi段和rw段中的数据其实可以是rw属性。

###RO/RW/ZI数据段

般而言，一个程序包括只读的代码段和可读写的数据段。

在ARM的集成开发环境中，只读的代码段和常量被称作**RO段(ReadOnly**)；

可读写的全局变量和静态变量被称作**RW段(ReadWrite)**；

RW段中要被初始化为零的变量被称为**ZI段(ZeroInit)**。


###连接

对于比较简单的情况，可以在ADS集成开发环境的ARM LINKER选项中指定**RO BASE和RW BASE**，告知连接器RO和RW的连接基地址。

对于复杂情况，如RO段被分成几部分并映射到存储空间的多个地方时，需要创建一个称为“**分布装载描述文件**”的文本文件，通知连接器把程序的某一部分连接在存储器的某个地址空间。需要指出的是，分布装载描述文件中的定义要按照系统重定向后的存储器分布情况进行。在引导程序完成初始化的任务后，应该把主程序转移到RAM中去运行，以加快系统的运行速度。


### |Image$$RO$$Base| |Image$$RO$$Limit| / |Image$$RW$$Base|/ |Image$$ZI$$Base| |Image$$ZI$$Limit|

这几个变量是编译器通知的，我们在 makefile文件中可以看到它们的值。它们指示了在运行域中各个输出段所处的地址空间

| Image$$RO$$Base| 就是ro段在运行域中的起始地址，
|Image$$RO$$Limit| 是ro段在运行域中的截止地址。
其它依次类推。

我们可以在linker的output中指定，在 simple模式中，ro base对应的就是| Image$$RO$$Base|，rw base 对应的是|Image$$RW$$Base|，由于rw和zi相连，|Image$$ZI$$Base| 就等于|Image$$ZI$$limit| .
其它的值都是编译器自动计算出来的。


## 启动代码

启动代码的搬运部分

- 将ro搬到sdram里，搬到的目的地址从 | Image$$RO$$Base| 开始，到|Image$$RO$$Limit|结束
- 搬rw段到sdram，目的地址从|Image$$RW$$Base| 开始，到|Image$$ZI$$Base|结束
- 将sdram zi初始化为0，地址从|Image$$ZI$$Base|到|Image$$ZI$$Limit|

##分散加载描述文件

Scatter file (分散加载描述文件)用于armlink的输入参数，他指定映像文件内部各区域的download与运行时位置。

Armlink将会根据scatter file生成一些区域相关的符号，他们是全局的供用户建立运行时环境时使用。

（注意：当使用了scatter file 时将不会生成以下符号 Image$$RW$$Base, Image$$RW$$Limit, Image$$RO$$Base, Image$$RO$$Limit, Image$$ZI$$Base, and Image$$ZI$$Limit）

### 什么时候使用scatter file

1 存在复杂的地址映射：例如代码和数据需要分开放在在多个区域。

2 存在多种存储器类型：例如包含 Flash,ROM,SDRAM,快速SRAM。我们根据代码与数据的特性把他们放在不同的存储器中，比如中断处理部分放在快速SRAM内部来提高响应速度，而把不常用到的代码放到速度比较慢的Flash内。

3 函数的地址固定定位：可以利用Scatter file实现把某个函数放在固定地址，而不管其应用程序是否已经改变或重新编译。

4 利用符号确定堆与堆栈：

5 内存映射的IO：采用scatter file可以实现把某个数据段放在精确的地指处。


##Reference

http://www.cnblogs.com/heart-of-eagle/archive/2011/04/28/2032240.html
