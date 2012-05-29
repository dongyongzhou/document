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
- Circular wait: cir	cular does not means deadlock.

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

## Thread Scheduling

**Scheduling:** deciding which threads are given access to resources


Two points:

- Which ready thread to schedule
- how many time should it work.

### Thread scheduling away from running

- IO request.
- time slice expired.
- fork a child
- interrupt.

programs alternate between bursts of CPU and I/O

### Scheduling Policy Goals/Criteria

1, Minimize Response Time

- Minimize elapsed time to do an operation (or job)
- Response time is what the user sees:

2, Maximize Throughput

- Maximize operations (or jobs) per second
- Throughput related to response time, but not identical:
Minimizing response time will lead to more context switching than if you only maximized throughput
- Two parts to maximizing throughput
    Minimize overhead (for example, context-switching)
    Efficient use of resources (CPU, disk, memory, etc)

3, Fairness

- Share CPU among users in some equitable way
- Fairness is not minimizing average response time:
Better average response time by making system less fair

### First-Come, First-Served (FCFS) Scheduling

“Run until done”

- In early systems, FCFS meant one program scheduled until done (including I/O)
- Now, means keep CPU until thread blocks 

Convoy effect: short process behind long process

FCFS Scheme: Potentially bad for short jobs!

- Depends on submit order

Simple 

Short jobs get stuck behind long ones 

### Round Robin (RR)

**Round Robin Scheme**

- Each process gets a small unit of CPU time (time quantum), usually 10-100 milliseconds
- After quantum expires, the process is preempted and added to the end of the ready queue
- n processes in ready queue and time quantum is q 
    Each process gets 1/n of the CPU time 
    In chunks of at most q time units 
    No process waits more than (n-1)q time units

**Performance**

- q large => FCFS
- q small => Interleaved
- q must be large with respect to context switch, otherwise overhead is too high (all overhead)

**Round-Robin Pros and Cons:**

- Better for short jobs, Fair (+)
- Context-switching time adds up for long jobs (-)

**How do you choose time slice**

Actual choices of timeslice:

- Initially, UNIX timeslice one second:

Worked ok when UNIX was used by one or two people.

What if three compilations going on? 3 seconds to echo each keystroke!

- In practice, need to balance short-job performance and long-job throughput:

Typical time slice today is between **10ms – 100ms**

Typical context-switching overhead is **0.1ms – 1ms**

Roughly **1% **overhead due to context-switching

**Comparisons between FCFS and Round Robin**

Assuming zero-cost context-switching time

when all jobs same length **Average response time** is much worse under RR!

Also: **Cache state** must be shared between all jobs with RR but can be devoted to each job with FCFS
**Total time for RR** longer even for zero-cost switch!
 
### Shortest Job First (SJF)/Shortest Remaining Time First (SRTF):

**Shortest Job First (SJF):**
Run whatever job has the least amount of computation to do

**Shortest Remaining Time First (SRTF):**
Preemptive version of SJF: if job arrives and has a shorter time to completion than the remaining time on the current job, immediately preempt CPU

**Comparison of SRTF with FCFS and RR**

- What if all jobs the same length?
SRTF becomes the same as FCFS (i.e., FCFS is best can do if all jobs the same length)

- What if jobs have varying length?
SRTF (and RR): short jobs not stuck behind long ones

**SRTF Pros & Cons**

- Optimal (average response time) (+)
- Hard to predict future (-)
- Unfair (-).Large jobs never get to run

can’t really know how long job will take
However, can use SRTF as a yardstick for measuring other policies

**Predicting the Length of the Next CPU Burst**

Adaptive: Changing policy based on past behavior
.CPU scheduling, in virtual memory, in file systems, etc.

Works because programs have predictable behavior

- If program was I/O bound in past, likely in future
- If computer behavior were random, wouldn’t help

SRTF with estimated burst length

Use an estimator function on previous bursts: 

Function f could be one of many different time series estimation schemes (Kalman filters, etc.)


### Multi-Level Feedback Scheduling

- First used in Cambridge Time Sharing System (CTSS)
- Multiple queues, each with different priority
- Each queue has its own scheduling algorithm


Adjust each job’s priority as follows (details vary)

- Job starts in highest priority queue
- If timeout expires, drop one level
- If timeout doesn’t expire, push up one level (or to top)


Scheduling must be done between the queues

- Fixed priority scheduling: 
Serve all from highest priority, then next priority, etc.
- Time slice:

### Lottery Scheduling:

Give each thread a priority-dependent number of tokens (short tasks  more tokens)

Reserve a minimum number of tokens for every thread to ensure forward progress/fairness

### Scheduling Fairness

Strict fixed-priority scheduling between queues is unfair 

Must give long-running jobs a fraction of the CPU even when there are shorter jobs to run

**implement** 

- Could give each queue some fraction of the CPU 
- Could increase priority of jobs that don’t get service

### Evaluate a Scheduling algorithm

- Deterministic modeling
- Queuing models
- Implementation/Simulation:


