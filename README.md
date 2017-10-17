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
- MMU: 1. Base and limit registers 2. Page table pointers, page protection, segmentation, TLB 3. Manipulating the hardware uses protected (privileged) operations

## Interrupt vs. Exceptions
1. Interrupts are caused by an external event (asynchronous) like device finishes I/O, timer expires
1. Exceptions are caused by executing instructions (synchronous) and CPU requires software intervention to handle a fault or trap, e.g., a page fault, write to a read-only page

## Interrupts
- Interrupts signal asynchronous events: IO hardware interrupts and timers

## I/O interrupts
- IRQs are mapped by Programmable Interrupt Controller to interrupt vectors, and passed to the CPU
- IPC is responsible for telling the CPU when and which device wishes to ‘interrupt’

Interrupt Vector Table (IVT)
- A data structure to associate interrupt requests with handlers.
- Store address of the handler and programmed by the OS
- called IDT in x86 with 256 entries

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

# Process
- the OS abstraction for execution
- the unit of execution and scheduling
- the dynamic execution context of a program
- a program in execution
- each process has own view of machine


Pros: simple programming, higher throughput and lower latency
1. increase CPU utilization - Overlap one process’s computation with another’s wait
vim     wait for input   wait for input gcc
2. reduce latency

## Kernel’s View of Processes
• A process contains all state for a program in execution
- An address space/code/data
- An execution stack -> the state of procedure calls
- The program counter -> the next instruction
- A set of general-purpose registers with current values
- A set of operating system resources

## Process Control Block (PCB)
- contains all of the info about a process
- Tracks state of the process: Running, ready (runnable), waiting, etc.
• Includes information necessary to run
- Registers, virtual memory mappings, etc.
- Open files (including memory mapped files)
- PCB is also maintained when the process is not running
• needed to restore the hardware to the same configuration it was in when the process was switched out
PS, PID, UserId, PC, Reg, VM addr, Open files

## Process state graph

## State queue
- The OS maintains a collection of queues that represent the state of all
processes in the system. Each PCB is queued on a state queue according to its current state
- As a process changes state, its PCB is unlinked from one queue and linked into another

## Context Switch
- Save PC and int reg always
- Save floating point or other special regs: only save if process used floating point
- Save condition codes
- Change virtual address translations

HW Optimization 1: don’t flush kernel’s own data from TLB
HW Optimization 2: use tag to avoid flushing any data

Usually causes more cache misses (switch working sets)

## Win
1. Creates and initializes a new PCB
1. Creates and initializes a new address space
1. Loads the program into the address space
1. Copies args into memory allocated in address space
1. Initializes the saved hardware context to start execution at main (or wherever specified in the file)
1. Places the PCB on the ready queue

## Unix - fork()
1. Same as win
1. Creates a new address space
1. Initializes the address space with a copy of the entire contents of the address
space of the parent
1. Initializes the kernel resources to point to the resources used by parent (e.g., open files)
1. Places the PCB on the ready queue

Returns the child’s PID to the parent, “0” to the child

## Unix - exec()
- Stops the current process
- Loads the program “prog” into the process’ address space
- Initializes hardware context and args for the new program
- Places the PCB onto the ready queue
- It does not create a new process

Pros
- Very useful when the child is cooperating with the parent or relies upon the parent’s data to accomplish its task
- Usage of manipulate file descriptors, set env var, reduce privileges, ...
- Simple interface

## Process Termination
- Essentially, free resources and terminate
- Terminate all threads
- Close open files, network connections
- Allocated memory (and VM pages out on disk)
- Remove PCB from kernel data structures, delete
- OS has to clean up.

## wait() or waitpid()
- pause until a child process has finished
- Unix: Every process must be “reaped” by a parent call wait(&status) to release the resources

>fork: Insufficient memory to create new process.
>
>exec: 1. Insufficient permission or no execute bit is set on the executable. 2. Path to executable is invalid. 3. Insufficient memory to execute.

The thread：a sequential execution stream within a process (PC, SP,
registers)
The process： the address space and general process attributes

## Thread
- Allocate TCB
- Allocate stack
- Build stack frame for base of stack - Put func, args on stack
- Put thread on ready queue

### Kernel level Thread - lightweight processes:
- All thread operations are implemented in the kernel
- The OS schedules all of the threads in the system
cons: 1. slower 2. one size fit all increases unessential overhead 3 high memory requirement
- Integrated with OS (informed scheduling)
- Slower to create, manipulate, synchronize

### User-level library
- One kernel thread per process
- Just lib func and library does thread context switch eg thread in java
cons: 1 take advantage of multicores 2 invisible to os st. os make poor decision
- Faster to create, manipulate, synchronize
- Not integrated with OS (uninformed scheduling

### yield()
context-switch between threads: cur thread to ready queue and switch to next
save context of cur, restore new, new becomes cur

## Sync:
1.shared resources(race condition) 2. coordinate operations(corruption)
no local(stack). global(static data) and dynamic(heap)

## Mutex: only one at a time
CS requirements: 1 mutex: only one at a time 2. progress: leave eventually and theads not in cs cant block others 3 no starvation 4 performance: little overhead for overall

Mechanisms:1. atomic r/w (starvation) 2. locks 3. semaphore 4. monitors 5. condition var

## Primitive
### Spin locks:
- Threads waiting to acquire lock spin in test-and-set loop
- Wastes CPU cycles
- Longer the CS, the longer the spin
- Greater the chance for lock holder to be interrupted

### Disabling Interrupts:
- Should not disable interrupts for long periods of time
- Can miss or delay important events (e.g., timer, I/O)

Higher level: Block waiters leave interrupts enabled within the critical section

## Deadlock
a set of processes if every process is waiting for an event that can be caused only by another process in the set.
- Mutual exclusion - Hold and wait - No preemption - Circular wait

Four approaches to dealing with deadlock:
- Ignore it – Living life on the edge
- Prevention – Make one of the four conditions impossible
- Avoidance – Banker’s Algorithm (control allocation)
- Detection and Recovery – Look for a cycle, preempt or abort

## semaphore: queue and var
Mutex semaphore (or binary semaphore)
- Represents single access to a resource
- Guarantees mutual exclusion to a critical section
Counting semaphore (or general semaphore)
- Represents a resource with many units available, or a resource that allows
certain kinds of unsynchronized concurrent access (e.g., reading)
- Multiple threads can pass the semaphore
- Number of threads determined by the semaphore “count” • mutex has count = 1, counting has count = N

## Monitors:
A monitor is a programming language construct that controls access to shared data

Access to the monitor is controlled by a lock
- wait() blocks the calling thread, and gives up the lock. To call wait, the thread has to be in the monitor (hence has lock). Semaphore: wait just blocks the thread on the queue

- signal() causes a waiting thread to wake up
If there is no waiting thread, the signal is lost. Semaphore: signal increases the semaphore count, allowing future entry even if no thread is waiting

Condition variables have no history

- Mesa monitors easier to use, more efficient: Fewer context switches, easy to support broadcast
- Hoare monitors leave less to chance • Easier to reason about the program

## CV
- Used by threads as a synchronization point to wait for events - Inside monitors, or outside with locks
- Wait – release monitor lock, wait for C/V to be signaled
- Signal – wakeup one waiting thread
- Broadcast – wakeup all waiting threads
