---
layout: master
title: TRACE32-ICD
---

# TRACE32-ICD 

## 目的

* 熟悉在线调试器的主要功能
* 掌握在线调试器TRACE32的调试功能
* 熟悉编写启动过程需要的批处理文件
* 熟练使用ICD调试系统
* 熟悉Simulator

## In-Circuit Debugger

Most CPUs provide an onchip debug system implemented in the CPU.

Most CPUs provide an onchip debug system implemented in the CPU. Typical examples
are the BDM interface from Motorola, the JTAG interface for the ARM7 or the JTAG
interface for the PowerPC family. The debug interface usually requires a few CPU pins that
are used for the communication between the onchip debug system and a third party
development tool. The onchip debug system provides the following basic features:

- Read/write memory
- Read/write CPU register
- Single step and real time execution
- Hardware breakpoints and trigger features (not supported by all CPUs)

## TRACE32-ICD功能

The In-Circuit Debugger TRACE32-ICD uses these basic features of the onchip debug
system to provide a powerful debug tool that offers:

- Easy high-level and assembler debugging
- Display of internal and external peripherals on a logical level
- Onchip break and trigger support
- RTOS awareness
- Flash programming
- Powerful script language
- Multiprocessor debugging


### TRACE32-ICD组件

1, A PODBUS interface to the host (PODPC, PODPAR or PODETH)

2, A Debug Module

3, A Debug Cable

The Debug Cable connects the Debug Module to the debug interface on your target.

4, A Clock Cable

By default TRACE32-ICD uses a fixed clock to run the debug interface. A clock range
from 100KHz up to 5 MHz can be used.

We also provide a clock cable to allow you the use of the divided CPU clock as clock for
the debug interface. **The relation between the CPU clock and the debug interface clock
is specific for your CPU**. Refer to your ICD Target Guide for detailed information. The
use of the divided CPU clock has the following advantages:

* The max. speed for the debug interface can be used. However we recommend
10MHz as the max. speed.

* The clock for the debug interface is automatically adapted if the CPU clock is
changed by your application program.


5, The TRACE32 debugger software


Host ------<PODBUS>-------- Debug Module ---------<Debug Interface>--------- Target


## TRACE32-ICD使用

### TRACE32-ICD安装

### TRACE32-ICD帮助

* Help button in toolbar
* Help->content/index...
* Command box: help

### TRACE32-ICD启动

要使In-Circuit调试器正常工作，必须要有一个能正常运转的目标机系统。

注意电源打开/关闭时的正确顺序：

- 打开：先调试器，再目标机。
- 关闭：先目标机，再调试器。

### TRACE32-ICD设置

Basic 设置启动软件属性

* Definition of an user specific configuration file.

By default the configuration file config.t32 in the system directory is used. 

The option -c allows you to define your own location and name for the configuration file.

* Definition of a working directory.

It is recommended not to work in the system directory.

* Definition of the start-up size of the application window.

### TRACE32-ICD Setup the Debug Environment

** Knowledge **

In order to set-up your debugger, you need some knowledge about your CPU and about
your target configuration. 

To be able to download your program including all symbol and debug information you also need some knowledge about your compiler.


CPU->System settings

1. The CPU specific settings.

Inform the debugger about the **CPU type** on your target, if an automatic detection of
the CPU is not possible. Select the correct CPU type from the pull down menu in the
field CPU.

    (Command: SYStem.CPU <CPU type>)

Set the **system options** in the option field corresponding to your target configuration
and application program.

    (Command: SYStem.Option <option>)

Set the **transfer clock** from the debug interface to your target. By default TRACE32-
ICD uses a fixed clock to run the debug interface. A clock range from 100KHz up to
5 MHz can be used. 

    (Command: SYStem.BdmClock)

We also provide a clock cable to allow you the use of the divided CPU clock as clock
for the debug interface. The relation between the CPU clock and the debug interface
clock is specific for your CPU. Refer to your ICD Target Manual for detailed
information. The use of the divided CPU clock has the following advantages:

• The max. speed for the debug interface can be used. However we recommend
10MHz as the max. speed.

• The clock for the debug interface is automatically adapted if the CPU clock is
changed by your application program.


2. Enter the debug mode.

Select the Up button in the Mode field to restart the CPU with debug mode enable.

    (Command: SYStem.Up)

The user interface establishes the communication to the target´s microprocessor. After this command
you should be able to access the registers.

3. Do the target specific settings.

The CPU is active now and you can initialize the CPU by writing to the special
function registers using the *Data.Set* command. E. g. some CPU need to set the
chip selects in order to access memory.

4. Load your application.

Load your application by using the command

    Data.LOAD.<option> <file_name>.

The option required for your compiler can be found in the ICD Target Manual in the
section Compiler.

If the file should be loaded to an Eeprom, the memory class EEPROM must be used to generate the
required programming sequence. 

    Example: d.load.b epromdata EEPROM:

For flash programming refer to the FLASH command group in the Reference book.
To display the source code the compiled program must be equipped with debug
information (frequently: compiler option “debug”). Then TRACE32 can load the
compiler output formats directly.

5. Initialize program counter and stackpointer: **Register.Set**

Many compilers add these settings in the startup code to the user program automatically.


It is recommended to write a batch job to set up the debugger to guarantee a proper startup
sequence.


### Save TRACE32 Configuration 

To save the window configuration for your TRACE32-ICD use *Store layout* from the
*Window* menu. Store layout generates a PRACTICE file, that includes all commands to
reactivate your complete window configuration automatically. Enter a filename for the
PRACTICE file into the File name field of the *Store* dialog box and push *Save* to
generate the PRACTICE file.


## TRACE32-ICD Batch Jobs

### Create a new batch file 

create *start.cmm* in your working directory by using the command

    PEDIT start.cmm.

TRACE32 has its own command language for batch jobs. It is called **PRACTICE** and it is very powerful (see
the PRACTICE User’s Guide and PRACTICE Reference for more information). All commands of the
TRACE32 development tools, commands for program flow, conditional commands and I/O commands are
allowed. The default extension for batch files is “.cmm”.

lso debugging of a PRACTICE program is supported. Look at the description in the PRACTICE User’s
Guide and PRACTICE Reference (commands: PLIST, PEDIT, PBREAK)

Enter the required commands and finish the batchjob by **ENDDO** and click the Save
button.

### Run Batchfile

Start the startup procedure by using Batchfile… in the File pulldown menu.

    FILE->run Batchfile


## TRACE32-ICD调试

### Open Running Codes

Open the **Data.List** window by using *Source/List* in the *View* menu. The program listing
around the program counter is displayed.

MODE:

- HLL: High Level Language.
- MIXed: assembler or mixed level.
 
Toggle the debug mode from MIXed to HLL by a click on the Mode button in the Data.List

### Steppgin run 


If you single step while an interrupt is pending, the next step will enter the interrupt routine.

If you want to single step your program without entering an interrupt routine use:

• SETUP.IMASKASM ON to disable the interrupts during single stepping on assembler
or mixed level.

• SETUP.IMASKHLL ON to disable the interrupts during single stepping on HLL level.


*Sigle step* : F2 / run->step / soft key

*Go Till Here*

*Step over* function call or subroutines.

*Go next* Go to the next code line written in the

*Go return* Go to the last instruction program listing. Useful e.g. to leave loops
of a function.

*Go Up* :Return to the caller function.

*Go* : Start the realtime emulation.

*Break* : Stop the realtime emulation

Reference book at the description of the commands Step.Change, Step.Till, Go.Change, Go.Till, Var.Step.Change,
Var.Step.Till, Var.Go.Change and Var.Go.Till.

Example: Var.Step.Till j>9 single steps the program until the
variable j becomes greater than 9.

### displays Stack

The *Var.Frame* window displays the function nesting for your application program. With
the option *LOCAL* the local variables of each function are displayed. When the option
*CALLER* is set, a few lines from the C-code are displayed to indicate where the function
was called.
 
VAR.FRAME /LOCAL /CALLER

### Display and Modify CPU Registers

Try to change a register value by *double-clicking* on the value you want to change. 

The *Register.Set* command for the selected register is displayed in the command line. You
only have to enter the new value and confirm it with return


Marked registers changed by the last steps in the Register window:

1, Click to the window header with the right mouse button.

The command, that was used to open the window is displayed in the command line
for modification. The window header becomes red.

2, Set the option /SpotLight and confirm the modification with return.

3, Execute a few single steps.

The registers changed by the last step are marked in dark red. The registers
changed in by the step before the last step are marked a little bit lighter etc. This
works up to a level of 4 step.


### How to Display and Modify the Special Function Registers

Open the Peripherals window to **display** your CPUs special function register:

- view->Peripherals
- per.view

**Modify** the contents of a special function register:

- By pressing the right mouse button and selecting one of the predefined logical
value from the pulldown menu.
- By a double-click to numeric values. A Data.Set command to change the register
contents is displayed in the command line. Enter the new value and confirm it with
return.

### How to Display and Modify Memory

To inspect an address range in the memory use the Data.dump window.

Select **Dump…** from the **View** menu. The Dump Memory dialog box is opened.

- Use the Browse button to browse through the symbol data base. Select a label by
a double-click and then confirm by pushing OK.
- Or enter the address directly in the Address field and push OK to open the
Data.dump window.
- Display a dump at address 0 by using the Memory Dump button
- Use the command line to display a memory dump,There a two different way to define an address range: 

     data.dump <start address>--<end address>
     data.dump <start address>++<offset>


Modify

The value at a memory address can be modified by **a double click**. A Data.Set
command for the selected address in displayed in the command line. Enter the new value
and confirm it with return.

### How to Set Breakpoints

**1, Software Breakpoints** 

The ICD debuggers use software breakpoints by default. 

When a software breakpoint is set to an instruction the code at this address is replaced by a special instruction e.g.
TRAP, that stops the realtime execution and returns the control to the onchip debug
system. *This method requires RAM at the break positions!* If you run your program
out of RAM the number of software breakpoints is unlimited.

If your program does not run in RAM, refer to *Breakpoints in ROM, FLASH or EEPROM*.

- *Set Breakpoint*

Back to the program. Doubleclick on the code line where you want to **set a program
breakpoint**. All code lines to which a program breakpoint is set are marked with a small
red bar.

To set a program breakpoint to a code line, that is not displayed select *Show Function*…
in the *Var* menu. Doubleclick to the function to display it and then set the breakpoint by a
doubleclick to the code line.

The second breakpoint type, that is available when software breakpoints are used, is a
**spot breakpoint**. A spot breakpoint is a watchpoint, that stops the program execution for a
short time to update all displayed information and then restarts the program execution.

*To set a spot breakpoint*, select the code line where it makes sense, that the displayed
information is updated. Press the right mouse button and select Spotpoint from the
pulldown menu.

*To watch for example all changes on the variable k*, select the variable by the mouse,
press the right mouse button and apply *Add to Watch Window* from the pulldown menu.


- *List Breakpoint*

Use *List* from the *Breakpoint* menu to display the information about all set breakpoints.

Break->list

Start the program execution with **Go**. If the program does not reach your breakpoint, you
can stop the program execution with **Break**.


- *remove Breakpoint*

You can remove the breakpoint by another doubleclick to the marked line or by toggling
the breakpoint in the Break.List window.


**2, Breakpoints in ROM, Flash, EEPROM** 

Most processor types (not 6833x and 6834x) provide a small number of onchip
breakpoints. These breakpoints are used by TRACE32-ICD to set program or spot
breakpoints even if the program doesn’t run in RAM. For information on the available
onchip breakpoints for your CPU refer to the Onchip Breakpoints (Overview)

Since the debugger uses software breakpoints by default, you must inform
the debugger that the onchip breakpoints should be used!

    MAP.BOnchip <address_range>

The command MAP.BOnchip indicates, that whenever a program or spot
breakpoint is set within the specified address range, the debugger should
use an onchip breakpoint.

**3, Breakpoints on Data Accesses** 

For most CPUs the provided onchip breakpoints can also be used by TRACE32-ICD to
**stop the program execution when a read or write access occurs to a specific address
location**. 

- To stop the program execution on a read access to a variable

select the variable with the cursor, press the write mouse button and select *Read from the Breakpoint… pulldown*.

- To stop the program execution on a write access to a variable

use *Breakpoints…* from the *Var* pulldown.

Browse through the *symbol* data base to find the variable, select it by a double
click, select *Write* in the Variable Breakpoint Set dialog box. Push OK to set the
breakpoint.

Or enter the variable name or the hll expression (e.g. to indicate only an element of
an array) in the Expression field of the Variable Breakpoint Set dialog box. Select
Write and push OK to set the breakpoint.

**4, Onchip Breakpoints (Overview)** 

• Onchip breakpoints

• Instruction breakpoints: onchip breakpoints that can be used for program
and spot breakpoints

• Data breakpoints: onchip breakpoints that can be used as read or
write breakpoints

### Display and Modify HLL Variables

To display HLL variables use the *Watch* command from the *Var* pulldown menu.

Select the variable by a double click from the symbol database

A quicker way to look at a variable is to mark the variable in the Data.List window by the
cursor and to press the right mouse button. From the Var pulldown menu select
*Add toWatch Window*.

If you want to display a more complex structure or an array in a separate window use
*View…* in the Var pulldown menu.

If you just want to **watch all variables accessed by the current program context** use
*Show Current Vars* from the *Var* pulldown menu and execute a view single steps.

You want to *inspect a variable and you are not sure about the spelling* open the symbol
browser to display all symbols stored in the internal symbol database.

*to modify a variable value*

 double click to the value. The appropriate Var.set command will be displayed in the command line. Enter the new value and confirm with
return.

### Format HLL-Variables

To adapt the display of a variable to your needs, select the variable name, press the right
mouse and select Format… from the Var pulldown menu.

Select Type to display the
variable with the complete
type information

If you display more complex HLL structures, select TREE in the Format field of the
Change Variable Format dialog box. This formatting allows to select the display for each
member of the structure by clicking on + or -.


## 参考资料

TRACE 32 In-Circuit Debugger Quick Installation and Tutorial
