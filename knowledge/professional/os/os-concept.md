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

A sequential execution stream within process (Sometimes called a “Lightweight process”)

including:

- Execution stack
- TCB: thread control block(including thread state, id etc)
- CPU registers(PC)

Share:

- Contents of memory (global variables, heap)
- I/O state (file system, network connections, etc)

#### communications between threads in the same Process
Share the memory resources allocated for Process.

### Inter-process Communication (IPC)

Message system – processes communicate with each other without resorting to shared variables
IPC facility provides two operations:

- send(message) – message size fixed or variable 
- receive(message)

Implementation

- physical (e.g., shared memory, hardware bus, syscall/trap)
- logical (e.g., logical properties)

### Address Spaces

The memory required to run the program is so called.
including: TEXT DATA bss stack heap 

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

1. Blocking on I/O
   :The act of requesting I/O implicitly yields the CPU
2. Waiting on a “signal” from other thread
:Thread asks to wait and thus yields the CPU
3. Thread executes a yield()

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

## Synchronization

### atomic 

### 
