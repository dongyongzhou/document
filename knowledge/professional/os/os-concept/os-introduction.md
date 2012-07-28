---
layout: master
title: Operating System Introduction
---

## Overview
 
## What does an Operating System do

Silberschatz and Gavin: “An OS is Similar to a government”

### Coordinator

- Manages all resources
- Settles conflicting requests for resources
- Prevent errors and improper use of the computer

### Facilitator (“useful” abstractions):

- Provides facilities/services that everyone needs
- Standard Libraries, Windowing systems
- Make application programming easier, faster, less error-prone

### Some features reflect both tasks:

- File system is needed by everyone (Facilitator) …
- … but File system must be protected (Traffic Cop)


## What is an Operating System

Most Likely:

- Memory Management
- I/O Management
- CPU Scheduling
- Synchronization / Mutual exclusion primitives
- Communications? (Does Email belong in OS?)
- Multitasking/multiprogramming?

What about?

- File System?
- Multimedia Support?
- User Interface?
- Internet Browser? 

## Sumaary


Operating systems provide a **virtual machine abstraction to handle diverse hardware**

Operating systems coordinate resources and protect users from each other

Operating systems **simplify application development** by providing standard services and abstractions

Operating systems can **provide an array of fault containment, fault tolerance, and fault recovery**



## How is the OS implemented

### Policy vs. Mechanism

- Policy: What do you want to do?
- Mechanism: How are you going to do it?
- Should be separated, since policies change 

### Algorithms used

Linear, Tree-based, Log Structured, etc…

### Event models used

Threads vs. event loops

### Backward compatibility issues

- Very important for Windows 2000/XP/Vista/…
- POSIX tries to help here

### System generation/configuration

How to make generic OS fit on specific hardware


## Virtual Machines

Software **emulation** of an abstract machine

- Make it look like hardware has features you want
- Programs from one hardware & OS on another one

Programming simplicity

- Each process thinks it has all memory/CPU time
- Each process thinks it owns all devices
- Different Devices appear to have same interface
- Device Interfaces more powerful than raw hardware

    - Bitmapped display  windowing system
    - Ethernet card  reliable, ordered, networking (TCP/IP)

Fault Isolation

- Processes unable to directly impact other processes
- Bugs cannot crash whole machine

Protection and Portability

- Java interface safe and stable across many platforms


## Uniprograming and Multiprograming

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

