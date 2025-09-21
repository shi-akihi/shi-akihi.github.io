---
layout: post
title: ICS1 Class Notes
date: '2025-09-21 15:12:15 +0000'
categories: [Computer Science, Basics]
tags: [SJTU, Computer Science, Notes, System, Introduction]
author: akihi
description: This is the note of the course SE2301(Introduction to Computer System 1) taken in the autumn semester in 2024. The course is based on the book CSAPP and mainly covers the first half part of the book. (Since taken during the course, the notes may seems messy. )
math: true
---

## Lecture 2

### The context of a compiler
- Preprocessor(to modified source program)
- Compiler(to assembly program)
- assembler(to relocatable object program)
- linker(to executable object program)

## Lecture 3
### Manipulating the Data
- General Boolean Algebra
- Mask Operation
- Logical Operations(all return 0 or 1, 0 for false and 1 for true)
- Shift Operation
	- Left Shift(fill with 0)
	- Right Shift(Arithmetic Shift VS Logical Shift)

### Byte Ordering
- Little Endian (the most significant byte has the lowest address)
- Big Endian

## Lecture 4

### Binary

Signed and Unsigned

### Casting
- Type Conversion(implicitly)
	- signed to unsigned by default
- Type Casting(explicitly)

## Lecture 5

### Register
- fastest storage unit in the computer
- using specific names(e.g. %rax, %rcx ......)
- most modern instructions can access registers only

### express operands in assembly
- immediate
	format: $imm
- registers(R->value)
	format: %rax
- virtual spaces(M->value)
	format: a linear array of bytes
$R[ ], M[ ]$ gives the value. 

### Indexed Addressing Mode

**Imm($r_b$, $r_i$, s)**
	Imm: constant(1, 2, 4, 8)
	$r_b$: base register
	$r_i$: index register
	s: scale(1, 2, 4, 8)
$$address = imm + R[r_b] + R[r_i] * s$$
$M[address]$ gives the value. 

### Assembly Instructions and their Execution mode

**Instruction**:
- basic operation
- operate on data in register
- move data between register and memory

Instructions are stored in virtual memory code area(read only). 

#### Mov Instruction

```
move src,dest
```

supported:
```
(immediate, register)   load
(memory, register)      load
(register, register)    load
(immediate, memory)     store
(register, memory)      store
```

We can only make the & operation to a variable if it is stored in the memory. 

Local variables are stored in stack. 

#### Stack

a pointer to the top of the stack: rsp

do the push and pop operation by operating rsp:
- push: decrease rsp & put value to the new space
- pop: store rsp to the register and add rsp

### Data Manipulation

#### Basic Operation

e.g. leaq S D: load the S address to the destination D

#### Specific Operation

Mul, div operations can only operation on %rax, %rdx


#### Conditional Codes
- a set of single-bit
- maintained in conditional code register(%eflag)
- describe the attributes of the most recent operation

e.g. 
- CF: Carry Flag
	used to detect overflow for unsigned operation
- OF: Overflow Flag 
	used to detect overflow for 2's-complement operation
- ZF: Zero Flag
	yield zero?
- SF: Sign Flag
	negative value?

**lea** instruction has no effect on conditional codes, which is used to perform some simple addition work. 

Conditional codes are implicitly set by basic operations. 

**Explicitly** set the conditional codes:
- cmp src2, src1(src1 - src2)
- test src2, src1(src2 & src1)

Conditional codes can't be accessed directly, but can be accessed by using **set commands**. 

### Control

**jump**: (go to)
- unconditional jump
	- direct jump: jmp label
	- indirect jump: jmp $*Operand$
- conditional jump: either jump or continue executing at the next instruction in the code sequence
	- all direct jump

**Jump Targets**: 
- PC-relative
- Absolute Address

## Lecture 6
### Control Constructs in C

- branch
- loop

#### Translate Conditional Branches

```c
	t = test-expr
	if (t) goto true;
	else-statement
	goto done
true:
	then-statement
done:
```

**cmove(conditional move)**: 
- source and destination can be 2, 4, 8 bytes
- destination must be a register
- source can be either memory or register

Note: Conditional Move instructions must suppose that there is no side effect. 

#### Do-while Translation
```c
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
```

#### While Loop Translation
```c
	go to test;
loop:
	body-statement
test:
	t = test-expr
	if (t)
		goto loop;
```

#### For Loop Translation

(first translate to while loop translation, then translate to do-while translation)

#### Switch Statements

**Jump Table**
- efficient implementation
- avoid long sequence of if-else statement
Approx. Translation:
```
target = JTab[op];
goto *target;
```

## Lecture 7

### Procedure Call

#### Basic Concept

- Terminology
	- caller
	- callee

#### Stack Frame Structure

- The portion of stack allocated for a procedure. 
- %rsp is the stack pointer to  identify the top of the frame
- The stack pointer can move when the procedure is executing. 

#### Invoke Callee

- Instruction
	- call label (direct)
	- call $*operand$ (indirect)
- Behavior description (by hardware)
	- save return address in the stack(end of caller frame; beginning of callee's frame)
	- jump to the entry of callee

#### Return to caller
- Instruction
	- ret
- Behavior description x

#### Passing data(register, stack)

- Specific registers to pass arguments(6)
- Specific register to keep the return value(%rax)

##### Passing Data: More than 6 Arguments

- Push by caller to stack
	- Saved in caller frame
	- From Nth to 7th(from right to left)
	- Just upon of return address
- Push by caller to register(6)

#### Registers: usage convention

##### Usage Convention
- Only once procedure can be active
- Partition registers between caller and callee
	- Caller-save registers
		- Callee can use these registers freely
		- Caller must restore them if it tries to use them after calling
	- Callee-save registers
		- Caller can use these registers freely
		- Callee must save them before using
		- Callee must restore them before return

## Lecture 8

#### Local variables: stack

- Allocation
	- below saved regs
	- move/sub %rsp
- De-allocation
	- move/add %rsp
- Usage
	- relative to %rsp

#### Recursion
In recursion, must use a callee-saved register. 

### Array

#### Array declaration

- allocate a contiguous region in memory

#### Starting address of an array

- The starting address of an array A is denoted as $X_A$ 
- Identifier A can be used as a pointer to the beginning of the array

### Nested Array

- Row major ordered in memory. 

## Lecture 9

### Fixed-Size Arrays

### Variable-Size Arrays
- Declare an array (either as a local variable or as an argument to a function)
- The dimensions of the array are determined

### Structure

### Union

#### The difference between structure and union:
- Structure stores like arrays in memory. 
- Union only allocates the largest memory size of its elements. 

### Alignment
#### Restrictions
- The address for some type of object must be a multiple of some value k. 
- Simplify the hardware design of the interface between the processor and the memory system. 

need to add gap or padding in the structure

## Lecture 10

### Buffer Overflow

mainly results from **out-of-bounds memory references**

The standards **gets** function may cause overflow. 

### Defense-1 Stack Randomization

- Address space layout randomization (ASLR)
	- each time a program is run 
	- different parts of the program are loaded into different regions of memory


### Defense-2 Stack Corruption Detection

Canary: randomly generated each time the program runs

**%fs:40**
- segmented addressing which appeared in 80286 and often used by thread local storage today
- It is marked as read only today

### Defense-3 Limiting Executable Code Regions

Code should be marked as "readable", "writable" and "executable". 

### Code Reuse Attack
- Return-oriented Programming
	- Find code gadgets in existed  code base
	- Push address of gadgets on the stack
	- Leverage 'ret' to connect code gadgets
	- No code injection

### Pointers
- Every pointer has a type
	- If the object has type T, a pointer to this object has type T*
	- Special void* type
	- malloc returns a generic pointer

Pointers are created with the & operator which is applied to lvalue expression. 

#### Pointer to Function

e.g. 
```c
int (*f)(int*)
```

### Supporting Variable-Size Stack Frames

Using %rbp to store the initial position of %rsp, and backtrace when leaving the function. 

## Midterm

### Byte Ordering

- **little endian(Intel)**
- big endian(Sun, IBM)

### Pointers & Arrays

### Type conversion & Casting

- casting: explicitly
- type conversion: implicitly(signed values implicitly cast to unsigned)

**Rule:** 
- short to long(zero extension / sign extension)

### Register & Memory

- Register
- virtual memory(user stack move from top to bottom(large address to small address))

Special Registers:
- %rax: return value
- %rsp: stack pointer
- %rip: program counter
- conditional code register: CF, OF, ZF, SF

### Addressing Mode

$Imm(r_b, r_i, s)$

### Mov Ops &  Data Manipulation

- Mov: cannot move between memory
- stack
	- push
	- pop
- Arithemetric operation (P129)

### Control Ops

- Set Eflags
	- cmp s2, s1(s2 - s1)
	- test s2, s1(s1&s2)
- Jump

### Function call

- save caller-save registers
- push rest actual arguments from right to left onto stack
- pass first six actual arguments
- save return address
- save callee-save registers
- allocate space for local variables
- ......

### Data struct

- struct: all the components are stored in contiguous region of memory
- union: all fields reference the same block
- alignment: the address for some type of the object must be a multiple of some value k(typically 2, 4, 8)

## Lecture 11

### Exception Flow Control 1

#### Fundamental abstractions

- Process
- Virtual memory
- Files(I/O)

#### Exception

##### Altering the Control Flow

New instructions can change control flow from user mode to kernel mode. 
- syscall / sysret

##### Parameter Passing for System Calls

- Up to 6 parameters are passed between two modes
- Caller saved registers need to be saved in user mode if necessary
	- %rcx, %r11 are destroyed by CPU for saving %rip and %rflags
- returns integer in %rax whether it is succeeded

**%rflags**: 
- More than conditional code
- Interrupt Enable Flag(IF)
	- whether to receive interrupts
- Trap Flag(TF)
	- single-step execution

##### Events

- Significant changes in the processor's state
	- results of execution of syscall/sysret instructions
	- data arrives from a disk or a network adapter

##### Exception

- a hardware mechanism transfers control to the kernel in response to some events

### Lecture 12

##### Exception Handler

- returns control to the current instruction
- returns control to the next instruction
- aborts the interrupted program

##### Exception Handler

- Each type of event has a unique exception number k
- Exception table entry k points to a function(exception handler)
- Handler k is called each time exception k occurs

exception numbers -> exception table->(code for exception handler)

One register to store exception table base address and one register to store exception number. 

- Exception handlers run in kernel mode
- Processor pushes necessary information onto kernel's stack(return address, RFLAGS, RSP)

**Page Faults**: CPU only load only a part of the data on the disk onto the memory. When the address on the virtual memory changes too much, the CPU will call the page fault and try to load the data onto the memory, if it failed, it will abort the program. 

**Virtual Memory**: make the program believe the memory is infinite. 

**Synchronous exceptions** 
- caused by events that occur as a result of executing an instruction
- e.g. trap, fault, abortion

**Asynchronous exceptions(interrupt)**
- caused by events external to the processor
- e.g. interrupt

##### How CPU Access IO Devices
- Memory-mapped IO
	- use physical address to access both DRAM or devices
- Port I/O

DMA: when DMA transfer completes, the disk controller notifies the CPU with an interrupt

##### When a New Process is Created

### Control flow

#### Context

- A context is the state that the kernel needs to restart an interrupted process. 
- Context contains:
	- the program's mode and data stored in memory 
	- value of PC, register file, status registers
	- user's stack, kernel's stack
	- environment variables
	- kernel data structures

## Lecture 13

#### Context Switching

- The multitasking(time slicing)
	- a mechanism that the OS performs
	- Higher-level form of exceptional control flow

**When will context switch happen?**

- The kernel is executing a system call on behalf of the user such as
	- read, sleep, etc. which will cause the calling process blocked
	- even if a system call block does not block, the kernel can decide the perform a context switch rather than return control to the calling process(e.g. page fault)
- As a result of an interrupt
	- Timer interrupt

#### Concurrency

#### Three states of a process
- Running
	- the process is either executing on the CPU or waiting to be called
- Stopped(blocked)
	- The execution of the process is suspended and will not be scheduled. 
- Terminated
	- The process is stopped permanently. 

- A running process becomes stop(SIGSTOP)
- A stopped process becomes running(SIGCONT)
- A process becomes terminated

#### System Call Error Handling

- Unix system-level functions encounter an error
	- typically return -1
	- set the global integer variable errno(to indicate what went wrong)

### Obtaining Process ID

- Process ID
	- PID
	- each process has a unique positive pid

#### Fork Function

- A parent process
	- creates a new running child process(different pid number)
	- by calling the fork function

- Called once
- Returns twice
	- return in the parent process(the pid of the child process)
	- return in the child process(0)

- Duplicate but separate address spaces
	- Any subsequent changes that  a parent or child makes to x is private and will not reflect in other process. 

### Zombie

- The kernel does not remove a terminated process from the system immediately
- The program is kept around in a terminated state until it is reaped by its parent

- A terminated process that has not yet been reaped is called a zombie

**reap zombie process**: 
1. parent process
2. init process(PID = 1; created by kernel during system initialization)

The zombie process will consume the system memory resources. 

## Lecture 14

### waitpid

 ```c
 pid_t waitpid(pid_t child, int *status, int options);
```


options: 
- 0: block the calling process until the child process terminates, returns the PID of the terminated child that caused waitpid to return
- WNOHANG: return immediately with a return value of 0 if none of the child processes in the waiting set has terminated yet
- WUNTRACED: suspend execution of the calling process until a child process terminated or stopped

status: (used to checking the exit status of a reaped child)
- WIFEXITED(status): true if terminated normally
- WEXITSTATUS(status): return the exit status of a normally terminated child
- WIFSIGNALED(status): true if the child process terminated because of an uncaught signal
- WTERMSIG(status): return the number of the signal(only when WIFSIGNALED is true)
- WIFSTOPPED
- WSTOPSIG

### Load and Running

```C
int execve(const char *filename, const char *argv[], const char *envp[]);
```

returns to the calling program only if there is an error


### Signal

Signal is a message that notifies a process, which corresponds to some kind of system event(low-level hardware or high-level program). 

#### Sending a signal

#### Receiving signal

#### Pending Signal

#### Blocking Signal

For each process, the kernel maintains the set of pending signals and the set of blocked signals. 

#### Kill
```c
int kill(pid_t pid, int sig);
```

- if pid > 0, send a signal to process pid
- if pid < 0, send a signal to process group abs(pid)

### Process Group

- Every process belongs to exactly one process group.

### Job 

- at most one foreground job and zero or more background jobs

#### Sending signals from keyboard

- ctrl+C will terminate all foreground job. 
- Ctrl+Z will suspend all foreground job. 

### Signal Function

- The same handler function can be used to different signal. 

Signal handlers can be interrupted. 

#### Blocking and unblocking signals

use sigprocmask function

### Concurrent Bugs

- keep handlers as simple as possible
- call only asyn-signal-safe functions in your handler
	- reentrant or can't be interrupted
- save and restore errno


## Lecture 15

Child process exit----set pending bit 1----calling process calls the signal handler and set the block bit 1----sigret to the OS and set the block bit 0

### Unix I/O

- All I/O devices are modeled as files

### File Type

- A regular file: contains arbitrary data
- A directory is a file consisting of
	- an array of links
	- each link maps a filename to a file
	- each directory contains at least two entries:(. / ..)
- A socket is a file that is used to communicate with another process across a network

### Open File

- How to know which inteeger should be return 
	- per process ID pool
- file cursor
	- per opened file data structure
- file information(属性)
	- per file metadata

#### Descriptor table(0, 1, 2 for stdin, stdout, stderr)

- Each process has its own separate descriptor table
#### File table(**shared by all processes**)

- current file position
- reference count(how many process points to the file table)

**Every fd number points to one file table.**

#### V-node table(**shared by all processes**)

- in main memory(cache the file metadata which on the disk for higher performance)


An application saves only the descriptor number. 

### Close File

- free the created memory resources
- default actions when terminate

### Reading and Writing Files

- read operation
	- copies m bytes from the file to memory(statrting at the current file position k and add m to k)
	- if the total bytes from k to the end of file is less than m, it triggers condition EOF(file has no  real EOF)


## Lecture 16

### Reading File Metadata

```c
int stat(const char *filename, struct stat *buf)
int fstat(int fd, struct stat *buf) // need the file to be opened first
```


### Reading Directory Contents

- directories are stored continuously regardless of the length of the string

### I/O Redirection

```c
int dup2(int oldfd, int newfd);
```

copy the struct pointed by oldfd to the new fd


#### How to restore stdout in Shell
```c
int dup(int oldfd);
```
- while the OS picks and returns a newfd for the duplication 


### Process Scheduling

- Mechanisms
	- low-level methods(that implement a needed piece of functionality)
	- examples: context switch
- Policies
	- Intelligence resides on top of the mechanisms
	- examples: scheduling policy(decide to run which process)

#### One Metric: turnaround time 

- turnaround time: the time from the job arrives in a system to it is finished
#### FIFO(first in, first out)

- convey effect
	- a number of relativelu-short potential consumers of a resource get queued behind a heavyweight resource consumer

#### SJF(shortest job first)

- run the shortest job first

#### STCF(shotest time-to-completion first)
- any time a new job enters the system

#### Another metric: response time

- response time: the time from when the job arrives in a system to the first time it is scheduled

##### Round Robin

- Methods
	- Run a job for a time slice
	- Switch to the next job in the run queue
	- Repeat until the jobs are finished
- Time Slice
	- Scheduling quantum
	- A multiple of the timer-interrupt period(10ms)
- Switching cost
	- Privilege switch, save and restore context
	- Indirect cost polluting TLB and the CPU pipelines
- Amortization
	- Do not switch too shortly
	- May increase the respond time
	- Must do some trade-off

### Incorporating I/O

- When a job initiates an I/O request
	- a system call is invoked
	- The job is blocked by kernel
	- Another job is scheduled to CPU
- When a job completes an I/O request
	- an interrupt is sent to CPU
	- The job becomes ready to run
		- The schedule determines which job to be scheduled

How to do the scheduling if the run-time of each job is unknown?
### MLFQ(Multi-Level Feedback Queue Scheduler)

- optimize the turnaround time without a prior knowledge of job length
- minimize the response time

#### Learning from History

- systems that learn from the past to predict the future
	-  the multi-level feedback queue
	- hardware branch predictors
	- caching algorithms
- Such approaches work when jobs have phases of behavior and are thus predictable
- Must be careful with such techniques

#### Basic Rules

- a number of distinct queues
	- each assigned a different priority level
- a job that is ready to run is on a single queue

1. if P(A) > P(B), A runs B doesn't
2. if P(A) == P(B), A & B run in RR
3. When a job enters the system, it is placed at the highest priority
4. If a job uses up an entire time slice while running, its priority is reduced
5. If a job gives up the CPU before the time slice is up, it stays at the same priority level
6. After some time period S, move all the jobs in the system to the topmost queue
7. (Refined) Once a job uses up its time allotment at a given level(regardless of how many times it has given up the CPU), its priority will be reduced

#### Potential Problems

- Starvation
- Gaming scheduler attack
- Programs may change its behavior over time 

#### Tuning MLFQ

- How to parameterize a MLFQ scheduler
- Most MLFQ variants allow for varying time-slice length across different queues

## Lecture 17

### Floating Point

#### Encoding Rational Numbers

- Form = $V=x \cdot 2^y$

Convert decimal float number into binary form : multiply 2 and round(乘2取整)

#### IEEE Floating Point Representation

- Numeric Form
$$V={(-1)}^s \cdot M \cdot 2^E$$
	- $s$  is the sign bit
	- M normally a fractional value in range $[1.0, 2.0)$
	- E weights value by power of two
- Single precision (32 bits) 1+8+23
- Double precision(64 bits) 1+11+52

**Exponent coded as biased value.**
- E = exp - bias
- Exp: unsigned value denoted by exp
- Bias: bias value(m is the number of exponent bits)
$$bias = 2^m - 1$$
##### Denormalized Value

- Condition: Exp = 00......0
- Values: 
	- E = 1 - Bias
	- m = 0.xx...x(bits of frac)

##### Special Values

- Exp = 111...1, frac = 00...0
- Represent value infinity(positive or negative)

- Exp = 111...1, frac != 00...0
- Not a number (NaN)
- Represents case when no numeric value can be determined

##### Dynamic Range

Distribution gets denser toward zero. 

#### Rounding Mode

- Round down(向下取整)
- Round up(向上取整)
- Round-toward-zero(去尾)
- Round-to-even
	- 末尾.5情况舍和入两种情况取偶数那个
	- Default Rounding Mode

#### Floating-Point Operations

- Conceptual View
	- First compute exact result
	- Make it fit into the desired precision

If m >= 2, shift m right and increment E. 
If m < 1, shift M left k positions, decrement E by k. 


## Lecture 18

### Floating Point

#### FP Code in procedures

- FP values are passed in XMM registers(Pointers and integers are passed in general-purpose registers)
- The mapping of arguments to registers depends on both their types and their ordering

#### FP Movement Operations

- Transfer values between memory and registers

#### Two-operand FP Conversion Operations

- Converting FP data to integers
	- Round values toward zero

#### Three-operand FP Conversion Operations

- Converting integer data to FPs
	- Ignore the second source

#### FP Arithmetic Operations

- One or two source operands, one destination operand
	- S1 can be XMM register or a memory location
	- S2 and D must be XMM registers

#### FP Bitwise Operations

#### FP Comparison Operations

