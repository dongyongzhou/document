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

*For the case of sleep.*

Since ints are disabled when you call sleep:

Responsibility of the next thread to re-enable ints

When the sleeping thread wakes up, returns to acquire and re-enables interrupts

Problem:
If the tread disable the interrupt. and crash, then The system will crash.


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


3. Monitor and Condition Variables

Problem is that semaphores .
They are used for both mutex and scheduling constraints

**Condition Variable:** a queue of threads waiting for something inside a critical section

**Key idea:** make it possible to go to sleep inside critical section by atomically releasing lock at time we go to sleep

Contrast to semaphores: Can’t wait inside critical section

**Operations**:

- Wait(&lock): Atomically release lock and go to sleep. Re-acquire lock later, before returning. 
- Signal(): Wake up one waiter, if any
- Broadcast(): Wake up all waiters

		Lock lock;
    	Condition dataready;
    	Queue queue;

		AddToQueue(item) {
    		lock.Acquire();	// Get Lock
    		queue.enqueue(item);	// Add item
    		dataready.signal();	// Signal any waiters
    		lock.Release();	// Release Lock
    	}

		RemoveFromQueue() {
    		lock.Acquire();	// Get Lock
    		while (queue.isEmpty()) {
    			dataready.wait(&lock); // If nothing, sleep
     		}
    		item = queue.dequeue();	// Get next item
    		lock.Release();	// Release Lock
    		return(item);
    	}


Use locks for mutual exclusion and condition variables for scheduling constraints

**Monitor**: a lock and zero or more condition variables for managing concurrent access to shared data

- Condition variable: a queue of threads waiting inside critical section for an event to happen
- Use condition variables to implement sched. constraints
- Three Operations: Wait(), Signal(), and Broadcast()


- Mutual Exclusion 
- Scheduling Constraints:A thread waiting for an event to happen in another thread



### For the Language support on Synchronization

* C: lock.aquire ->action ->lock.release.  
Pretty straightforward synchronization
Just make sure you know all the code paths out of a critical section

* C++ :Languages that support exceptions are problematic (easy to make a non-local exit without releasing lock).  almost the same, but for on exception, should catch and lock.release.
* Java: Every object has an associated lock which gets automatically   acquired and released on entry and exit from a synchronized method

	class Account {
		private int balance;
		// object constructor
		public Account (int initialBalance) {
			balance = initialBalance;
		}
		public synchronized int getBalance() {
			return balance;
		}
		public synchronized void deposit(int amount) {
			balance += amount;
		}
	}

Since every Java object has an associated lock, this type of statement acquires and releases the object’s lock on entry and exit of the code block

In addition to a lock, every object has a single condition variable associated with it

How to wait inside a synchronization method of block:

    void wait();
    void wait(long timeout); // Wait for timeout
    void wait(long timeout, int nanoseconds); //variant

How to signal in a synchronized method or block:

    void notify();	// wakes up oldest waiter
    void notifyAll(); // like broadcast, wakes everyone


### Resource

Resources – passive entities needed by threads to do their work. CPU time, disk space, memory

Two types of resources:

- Preemptable – can take it away.CPU, Embedded security chip
- Non-preemptable – must leave it with the thread.Disk space, printer, chunk of virtual address space
Critical section 

Resources may require *exclusive access* or may be *sharable*

Read-only files are typically sharable

Printers are not sharable during time of printing

One of the major tasks of an operating system is to manage resources


### Starvation vs Deadlock

**Starvation**: thread waits indefinitely

**Deadlock**: circular waiting for resources

Deadlock => Starvation but not vice versa

- Starvation can end (but doesn’t have to)
- Deadlock can’t end without external intervention

Deadlock not always deterministic 

Deadlock won’t always happen with the code
Have to have exactly the right timing (“wrong” timing?)

Deadlocks occur with *multiple resources*
Means you can’t decompose the problem,
Can’t solve deadlock for each resource independently

**Four requirements for Deadlock**

- Mutual exclusion
- Hold and wait
- No preemption
- Circular wait

**Methods for Handling Deadlocks**

Allow system to enter deadlock and then recover

- Requires deadlock detection algorithm
- Some technique for forcibly preempting resources and/or terminating tasks

Deadlock prevention: ensure that system will never enter a deadlock

- Need to monitor all lock acquisitions
- Selectively deny those that might lead to deadlock

Ignore the problem and pretend that deadlocks never occur in the system
Used by most operating systems, including UNIX


**Deadlock Detection Algorithm**

See if tasks can eventually terminate on their own


**Techniques for Preventing Deadlock**

- Infinite resources
Not very realistic

- No Sharing of resources (totally independent threads)
Not very realistic

- Don’t allow waiting 

- Make all threads request everything they’ll need at the beginning

- Force all threads to request resources in a particular order preventing any cyclic use of resources


*Banker’s Algorithm for Preventing Deadlock*

Toward right idea: 

- State maximum resource needs in advance
- Allow particular thread to proceed if:
	(available resources - #requested) >= max 
 remaining that might be needed by any thread
