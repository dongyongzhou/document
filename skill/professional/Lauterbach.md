---
layout: master
title: Lauterbach
---

# Lauterbach 

##1 Overview

Introduction to Lauterbach JTAG tool

- tool installation
- configuration
- license issues
- cmm files
- TURBO mode 
- loading code
- stepping through code
- examining and modifying register and data memory
- using breakpoints 
- Lauterbach different hardware configurations
- "On Host" option

##2 tool installation


###2.1 T32start

##3 Help

###3.1 Index

###3.2 command

"system.option.enreset " + F1

###3.2 setting

"system.option.enreset " + enter


##4 COMMAND


###4.1 symbol.list.source

###4.2 data.testlist x-xxx

##5 configuration

##5.1 SYStem.JtagClock

Selects the JTAG port frequency (TCK) used by the debugger to communicate with the processor. This influences e.g. the download speed. 

It could be required to reduce the JTAG frequency if there are buffers, additional loads or high capacities on the JTAG lines or if VTREF is very low. 

A very high frequency will not work on all systems and will result in an erroneous data transfer. Therefore we recommend to use the default setting if possible.

## Reference


