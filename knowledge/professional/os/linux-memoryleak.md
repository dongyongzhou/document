---
layout: master
title: Linux Kernel Memory Leak
---

## Overview

Lets disscuss How to detect Kernel Memory leak

## Basic Knowledge

### Memory Leak

- In computer science, occurs when a computer program acquires memory but fails to release it back to the operating system.

- In object-oriented programming, a memory leak may happen when an object is stored in memory but cannot be accessed by the running code.

A memory leak has symptoms similar to a number of other problems and generally can only be diagnosed by a programmer with access to the program source code.

### Memory mode for C++ and Java

- local variable: saved in Stack(fast for access)
- Object (new)  : saved in Heap
- Object reference (new)  : saved in Stack(fast for access)


![](http://p.blog.csdn.net/images/p_blog_csdn_net/IloveAgile/EntryImages/20090122/memory.JPG)
## How does Memory Leak come

- （1）No Referenced Memory

   (C++ only) new->delete， malloc()->free()

   (Java ) No free or delete needed. garbage collection(GC)

- （2）No free Objects/Pointers (C++ and Java)

     
     String[] sa = new String[9999999];
     for (int i = 0; i < 9999999; i++){
       String s = new String(“adfasdfadsfas…adfasdfa”); //a 1MB size string…
       sa[i] = s;
     }

- （3）No Limited Storage (C++ and Java)

    Vector, hashtable, hashmap, map, arraylist and String StringBuffer


## How to detect Memory Leak


- Long Run
- special Case


Tools 


（1）Purify
   一般For C Code
  （２）Heap Analyzer
    可以For Java Code
  （3）Java Dump


## Summary

## Reference

...
