---
layout: master
title: TRACE32-Debugger
---

# TRACE32-Debugger 

## Memory Display and Modification

### Memory Classes

    Memory Class Description
    - P Program Memory
    - D Data Memory
    
    - S Supervisor Memory (privileged access)
    - U User Memory (non-privileged access)    - 
        not yet implemented; privileged access will be performed
    - R ARM Code (32-bit)
    - T Thumb Code (16-bit)
    - J Java Code (8-bit)
    
    - Z Secure Mode (TrustZone devices)
    - N Non-Secure Mode (TrustZone devices)
    
    - A Absolute addressing (physical address)
    - ICE ICE Breaker Register (debug register; ARM7, ARM9)
    - C14 Coprocessor 14 Register (debug register; ARM10, ARM11)
    - C15 Coprocessor 15 Register (if implemented)
    - ETM Embedded Trace Macrocell Registers (if implemented)
    - DAP Memory access via Debug Access Port (CoreSight, Cortex). The access
          port specified by SYStem.MultiCore DEBUGACCESSPORT is used
          which is typically the APB-AP.
    - VM Virtual Memory (memory on the debug system)
    - USR Access to Special Memory via User Defined Access Routines
    - E Run-time memory access
        (see SYStem.CpuAccess and SYStem.MemAccess)



## 参考资料

- debugger_arm.pdf
- training_debugger.pdf

