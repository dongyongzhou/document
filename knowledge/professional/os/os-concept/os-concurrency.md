---
layout: master
title: Operating System Concurrency
---

## Overview

### The Basic Problem of Concurrency

The basic problem of concurrency involves **resources**:

- Hardware: single CPU, single DRAM, single I/O devices
- Multiprogramming API: processes think they have exclusive access to shared resources

OS has to **coordinate** all activity

- Multiple processes, I/O interrupts, …
- How can it keep all these things straight?

Basic Idea: Use Virtual Machine abstraction

- Simple machine abstraction for processes
- Multiplex these abstract machines

Dijkstra did this for the “THE system”

- Few thousand lines vs 1 million lines in OS 360 (1K bugs)

### Uniprogramming: one thread at a time

- MS/DOS, early Macintosh, Batch processing
- Easier for operating system builder
- Get rid of concurrency (only one thread accessing resources!)

### Multiprogramming: more than one thread at a time

- Multics, UNIX/Linux, OS/2, Windows NT – 7, Mac OS X
- Often called “multitasking”, but multitasking has other meanings (talk about this later)


### Challenges of Multiprograming

- Each applications wants to own the machine  **virtual machine abstraction**

- Applications **compete with each other for resources**

    - Need to arbitrate access to shared resources  **concurrency**
    - Need to protect applications from each other  **protection**

- Applications need to communicate/cooperate with each other  **concurrency**

### illusion of multiple processors

Assume a single processor.  How do we provide the illusion of multiple processors?

- Multiplex in time!

Each virtual “CPU” needs **a structure** to hold:

- Program Counter (PC), Stack Pointer (SP)
- Registers (Integer, Floating point, others…?)

How switch from one CPU to the next?

- Save PC, SP, and registers in current state block
- Load PC, SP, and registers from new state block

What triggers switch?

- Timer, voluntary yield, I/O, other things

### Properties of this simple multiprogramming technique

All virtual CPUs share same non-CPU resources

- I/O devices the same
- Memory the same

Consequence of sharing:

- Each thread can **access the data of every other thread** (good for **sharing**, bad for protection)
- Threads can **share instructions**(good for sharing, bad for protection)
- Can threads overwrite OS functions? 

This (unprotected) model common in:

- Embedded applications
- Windows 3.1/Early Macintosh (switch only with yield)
- Windows 95—ME? (switch with both yield and timer)


## Processes

**Process**: unit of resource allocation and execution

- Owns memory (address space)
- Owns file descriptors, file system context, …
- Encapsulate one or more threads sharing process resources


- A program in execution.
- A unit of dispatching for execution.
- A unit of resource ownership

including: 

- Address space,
- PC,
- SP,
- FP,
- other registers,
- I/O resources.

### Process and Program

More to a process than just a program:

- Program is just part of the process state

Less to a process than a program:

- A program can invoke more than one process
- cc starts up cpp, cc1, cc2, as, and ld

### Why processes? 

- Navigate fundamental tradeoff between protection and efficiency
- Processes provides memory protection while threads don’t (share a process memory)
- Threads more efficient than processes (later)

Application instance consists of one or more processes 

### Traditional UNIX Process

Process: Operating system abstraction to represent what is needed to run a single program

- Often called a “HeavyWeight Process”
- Formally: a single, sequential stream of execution in its own address space

Two parts:

- Sequential Program Execution Stream

    - Code executed as a single, sequential stream of execution (i.e., thread)
    - Includes State of CPU registers

- Protected Resources:

    - Main Memory State (contents of Address Space)
    - I/O state (i.e. file descriptors)

Important: There is no concurrency in a heavyweight process

### How do we Multiplex Processes?

The current state of process held in a **process control block** (PCB):process state, process number, program counter, registers, opened files,...

- This is a “snapshot” of the execution and protection environment
- Only one PCB active at a time

Give out CPU time to different processes (**Scheduling**):

- Only one process “running” at a time
- Give more time to important processes

Give pieces of resources to different processes (**Protection**):

- Controlled access to non-CPU resources
- Sample mechanisms: 

    - Memory Mapping: Give each process their own address space
    - Kernel/User duality: Arbitrary multiplexing of I/O through system calls

### CPU Switch From Process to Process

This is also called a “context switch”

Code executed in kernel above is **overhead**
 
- Overhead sets minimum practical switching time
- Less overhead with SMT/Hyperthreading, but… contention for resources instead


### Process State

As a process executes, it changes state

- new:  The process is being created
- ready:  The process is waiting to run(admitted from new, and interrupt from running)
- running:  Instructions are being executed(schedular dispatch from ready)
- waiting:  Process waiting for some event to occur(I/O or event wait from running)
- terminated:  The process has finished execution(exit from running)

### communications between Processes

1.Shared-Memory Mapping
  Accomplished by mapping addresses to common DRAM
  Read and Write through memory
 
  How could proc2 knows proc1 finished writing?
  *Really low overhead communication
  *Introduces complex synchronization problems
  **Inter-process Communication (IPC)
2.Message Passing
  send() and receive() messages
  Works across network

### Process Scheduling

PCBs move from queue to queue as they change state

- Decisions about which order to remove from queues are Scheduling decisions
- Many algorithms possible 

PCBs from running to ready queue

- I/O request-> I/O queue->I/O->ready queue
- time slice expired
- fork a child
- waiting for an interrupt

### create a process

#### Must construct **new PCB** 

Inexpensive

#### Must set up **new page tables** for address space

More expensive

#### Copy **data from parent process**? (Unix fork() )

- Semantics of Unix fork() are that the child process gets a complete copy of the parent memory and I/O state
- Originally very expensive
- Much less expensive with “copy on write”

- Copy I/O state (file handles, etc)

Medium expense

### Multiple Processes Collaborate on a Task

High **Creation/memory Overhead**

(Relatively) High **Context-Switch Overhead**

Need **Communication mechanism**:

- Separate Address Spaces Isolates Processes
- Shared-Memory Mapping

    - Accomplished by mapping addresses to common DRAM
    - Read and Write through memory

- Message Passing

    - send() and receive() messages
    - Works across network

### Shared Memory Communication

Communication occurs by “simply” **reading/writing to shared address page**

- Really low overhead communication
- Introduces complex synchronization problems

## Threads

A sequential execution stream within process (Sometimes called a “Lightweight process”)

- Independent Fetch/Decode/Execute loop
- Unit of scheduling
- Operating in some address space

including:

- Execution stack
- TCB: thread control block(including thread state, id etc)
- CPU registers(PC)

Share:

- Contents of memory (global variables, heap)
- I/O state (file system, network connections, etc)

### protect threads from one another

Protection of memory

- Every task does not have access to all memory

Protection of I/O devices

- Every task does not have access to every device

Protection of Access to Processor: preemptive switching from task to task

- Use of timer
- Must not be possible to disable timer from usercode

### communications between threads in the same Process

Share the memory resources allocated for Process.










## Inter-process Communication (IPC)

Message system – processes communicate with each other without resorting to shared variables
IPC facility provides two operations:

- send(message) – message size fixed or variable 
- receive(message)

Implementation

- physical (e.g., shared memory, hardware bus, syscall/trap)
- logical (e.g., logical properties)

## Address Spaces

The memory required to run the program is so called.

Address space: the set of accessible addresses + associated states:

- For a 32-bit processor there are 232 = 4 billion addresses


including: TEXT DATA bss stack heap 

### What happens when you read or write to an address?

- Perhaps nothing
- Perhaps acts like regular memory
- Perhaps ignores writes
- Perhaps causes I/O operation
(Memory-mapped I/O)
- Perhaps causes exception (fault)

## Thread Dispatching

at least one thread in a process.

### Thread Control Block (TCB)
- Execution State: CPU registers, program counter, pointer to stack
- Scheduling info: State, priority, CPU time
- Various Pointers (for implementing scheduling queues)
- Pointer to enclosing process? (PCB)?
- Etc (add stuff as you find a need)

### Queues

### dispatcher

#### get control back
- Internal events: thread returns control voluntarily

1, Blocking on I/O
   :The act of requesting I/O implicitly yields the CPU
2, Waiting on a “signal” from other thread
:Thread asks to wait and thus yields the CPU
3, Thread executes a yield()

- External events: thread gets preempted

1. timer interrupt 

### Questions

- How do we position stacks relative to each other?
- What maximum size should we choose for the stacks?
- What happens if threads violate this?
- How might you catch violations?
- are Thread queues in process`s resource?
- where is the TCB of the thread that are in running state?
- Unix nice()?
- where is the dispatch loop

## Cooperating Threands

### Threads 

### Question

+ Why disable the globle interupt on entering and exsting handler?

Protecting the behavior of saving or restoring the field of all the register, Some CPU might make these atomic(save all or save none).

+ What does ThreadHouseKeeping do?

Notice waitToBeDestroyed thread and deallocate the TCB and stack.

+ Why allow cooperating threads?

- share resource
- speedup. overlap I/O and computation, multipleprocessors
- modularity: 



