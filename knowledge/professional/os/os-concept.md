---
layout: master
title: Operating System Concept
---

# Operating System Concept
 
## What is Operating System

## Operating Systems Structures

## Concurrency.Processes.Threads.and.Address.Spaces

### Processes

A program in execution.
A unit of dispatching for execution.
A unit of resource ownership
including: 
Address space,
PC,
SP,
FP,
other registers,
I/O resources.

#### communications between Processes

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

### Threads

A sequential execution stream within process (Sometimes called a ¡°Lightweight process¡±)
including: 
-Execution stack
-TCB: thread control block(including thread state, id etc)
-CPU registers(PC)
Share:
-Contents of memory (global variables, heap)
-I/O state (file system, network connections, etc)

#### communications between threads in the same Process
Share the memory resources allocated for Process.

### Inter-process Communication (IPC)

Message system ¨C processes communicate with each other without resorting to shared variables
IPC facility provides two operations:
-send(message) ¨C message size fixed or variable 
-receive(message)

Implementation
physical (e.g., shared memory, hardware bus, syscall/trap)
logical (e.g., logical properties)

### Address Spaces

The memory required to run the program is so called.
including: TEXT DATA bss stack heap 

## Thread Dispatching


### 
