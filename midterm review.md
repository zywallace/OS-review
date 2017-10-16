# Basics
OS: Layer between applications and hardware
## Benefits to HW
1. Manage HW resources: cpu, memory, disk, communication and IO
1. Provides abstractions: Processes, threads, VM, filesys
1. Mediate accesses from different applications- who at what point and how long

## Benefits to APP
1. Simpler
1. Device independent
1. Portable

## VM interface
An OS defined, logical environment and each program thinks it owns the computer
### Protection
Prevents one process/user from clobbering another
### Provides sharing
1. Multitasking (time slicing)
1. Communication
1. Shared implementations of common facilities like file system

## A Primitive OS
One program at a time and no bad users or programs
cons: poor utilization of both pc and human

## Multitasking:
More than one process can be running at once and when one process blocks (waiting for disk, network, user input, etc.) run another.

### Mechanism: context-switch
When one process resumes, it can continue from last execution point.

#### Problems: ill-behaved process
1. go into infinite loop and never relinquish CPU
1. scribble over other processes’ memory to make them fail

#### Solutions:
1. scheduling: fair sharing, take CPU away from looping process
1. virtual memory: protect process’s memory from one another

# Architecture
## P
User mode
- Limited privileges
- Only those granted by the operating system kernel • Kernel mode
- Execution with the full privileges of the hardware
- Read/write to any memory, access any I/O device, read/write any disk sector,
send/read any packet

Protected Instructions
• A subset of instructions restricted to use only by the OS - Known as protected (privileged) instructions
• Only the operating system can ...
- Directly access I/O devices (disks, printers, etc.)
• Security,fairness(why?)
- Manipulate memory management state
• Pagetablepointers,pageprotection,TLBmanagement,etc. - Manipulate protected control registers
• Kernelmode,interruptlevel - Halt instruction (why?)
