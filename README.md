# Basics
OS: Layer between applications and hardware
## Benefits to HW
- Manage HW resources: cpu, memory, disk, communication and IO
- Provides abstractions: Processes, threads, VM, filesys
- Mediate accesses from different applications- who at what point and how long

## Benefits to APP
- Simpler
- Device independent
- Portable

## VM interface
An OS defined, logical environment and each program thinks it owns the computer
### Protection key goals
Prevents one process/user from clobbering another
### Provides sharing
- Multitasking (time slicing)
- Communication
- Shared implementations of common facilities like file system

## A Primitive OS
One program at a time and no bad users or programs
cons: poor utilization of both pc and human

## Multitasking:
More than one process can be running at once and when one process blocks (waiting for disk, network, user input, etc.) run another.

### Mechanism: context-switch
When one process resumes, it can continue from last execution point.

#### Problems: ill-behaved process
- go into infinite loop and never relinquish CPU
- scribble over other processes’ memory to make them fail

#### Solutions:
- scheduling: fair sharing, take CPU away from looping process
- virtual memory: protect process’s memory from one another

# Architecture
## Types of Arch Support
1. Manipulating privileged machine state
1. Generating and handling “events”
1. Mechanisms to handle concurrency

## User mode
- Limited privileges
- Only those granted by the operating system kernel

## Kernel mode
- Execution with the full privileges of the hardware
- Read/write to any memory, access any I/O device, read/write any disk sector, send/read any packet

One mode: Fetch->Decode->Execute

Dual-Mode Operation
- Check if a instruction is allowed to be executed
- Change mode upon certain instructions

## Privileged Instructions
A subset of instructions restricted to use only by the OS
- Directly access I/O devices
- Manipulate memory management state
- Manipulate protected control registers
- Halt instruction

## Memory Protection
- OS must be able to protect programs from others and itself

### Simple Memory Protection: bound check
`if(< base || > bound) raises exception`

cons:
- Inflexible: fix allocation, difficult to expand head and stack
- Memory sharing and fragmentation
- Physical memory: require changes to memory instructions each time the program is loaded
Processor

### Virtual Address
Programs refer to memory by virtual addresses - start from 0
like “owning” the entire memory address space

The virtual address is translated to physical address
- upon each memory access
- done in hardware using a table - setup by the OS


## Interrupt vs. Exceptions
1. Interrupts are caused by an external event (asynchronous) like device finishes I/O, timer expires
1. Exceptions are caused by executing instructions (synchronous) and CPU requires software intervention to handle a fault or trap, e.g., a page fault, write to a read-only page

## Interrupts
- Interrupts signal asynchronous events
- Indicates some device needs services - I/O hardware interrupts
- Software and hardware timers

Interrupt Vector Table (IVT)
- A data structure to associate interrupt requests with handlers.
- Store address of the handler and programmed by the OS
- called IDT in x86 with 256 entries

## I/O interrupts
- IRQs are mapped by Programmable Interrupt Controller to interrupt vectors, and passed to the CPU
- IPC is responsible for telling the CPU when and which device wishes to ‘interrupt’
> After the hardware saves the old stack pointer, the old program counter, and other registers, it switches to the kernel stack at the beginning of the (assembly language) interrupt handler stub. The interrupt handler stub must then save (and restore) the remaining register state, set up (tear down) the interrupt handler stack frame, and call into (return from) the interrupt handler, and execute the handler. The handler itself is device specific, but in general it will at least read control registers off the device to determine what operation has completed.
>
> The CPU initiates a trap. Usually an I/O device initiates an interrupt. Traps are often used to catch arithmetic conditions (e.g. over- flow, divide by zero). Interrupts are used to interrupt the CPU and run an interrupt service routine to handle some externally generated condition.

## Trap
- An intentional software-generated exception
- The main mechanism for programs to interact with the OS x86 int to cause a trap
- Handler for trap is defined in interrupt vector table

### System Call Trap
For a user program to “call” OS service
- Causes an exception, which vectors to a kernel handler
- Passes a parameter determining the system routine to call
- Saves caller state (PC, regs, mode) so it can be restored
- Returning from system call restores this state
Requires architectural support to: restore saved state, reset mode, resume execution

## Faults
Hardware detects and reports “exceptional” conditions - Page fault, unaligned access, divide by zero

### Handling Faults
#### Fixing
- “Fix” the exceptional condition and return to the faulting context
- Page faults cause the OS to place the missing page into memory
- Fault handler resets PC of faulting context to re-execute instruction that caused the page fault

#### Notifying the process
- Fault handler changes the saved context to transfer control to a user-mode
handler on return from fault
- Handler must be registered with OS

#### killing the process
- Program fault with no registered handler
- Halt process, write process state to file, destroy process
- In Unix, the default action for many signals
