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

**Synchronization**: using *atomic operations* to ensure cooperation between threads

### mutual exclusion 

**Mutual Exclusion**: ensuring that only one thread does a particular thing at a time

### critial section

**Critical Section**: piece of code that only one thread can execute at once

- Critical section and mutual exclusion are two ways of describing the same thing
- Critical section defines sharing granularity 

### atomic operation

do it all, or none

- Hardware atomic. disable/enable interrupt, Load/store, swap . etc.
- software. critial section/mutual exclusion: (using locks)

**How can we build multi-instruction atomic operations?**

Lock 

Recall: dispatcher gets control in two ways. 

- Internal: Thread does something to relinquish the CPU
- External: Interrupts cause dispatcher to take CPU

On a uniprocessor, can avoid context-switching by:

- Avoiding internal events (although virtual memory tricky)
- Preventing external events by disabling interrupts

1. Lock

**Lock**: prevents someone from accessing something

- Lock before entering critical section (e.g., before accessing shared data)
- Unlock when leaving, after accessing shared data
- Wait if locked

Important idea: all synchronization involves waiting

Should sleep if waiting for long time

**Implementation of Locks by Disabling Interrupts**

Key idea: maintain a lock variable and impose mutual exclusion only during operations on that variable

**Implementing Locks with test&set**

Simple solution:

		int value = 0; // Free
		Acquire() {		while (test&set(value)); // while busy	}
		Release() {		value = 0;	}
Simple explanation:

If lock is free, test&set reads 0 and sets value=1, so lock is now busy.  It returns 0 so while exits

If lock is busy, test&set reads 1 and sets value=1 (no change). It returns 1, so while loop continues

When we set value = 0, someone else can get lock

2. Semaphores

**Definition**: a Semaphore has a non-negative integer value and supports the following two operations:

- P(): an atomic operation that waits for semaphore to become positive, then decrements it by 1 
*Think of this as the wait() operation*

= V(): an atomic operation that increments the semaphore by 1, waking up a waiting P, if any
*This of this as the signal() operation*

**Use case:**

- Mutual Exclusion 
- Scheduling Constraints


### 
