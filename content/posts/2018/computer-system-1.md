---
title: "Computer System 1"
date: 2018-05-23T23:54:04
categories:
- Things I Learned
tags:
- System
---

## Hardware Organization of A System ##

A typical system has those hadrware below:
<!-- more -->

- Buses: Running throughout the system is a collection of electrical conduits called buses that carry bytes of ingormation
back and forth betweent the components. Buses are typically designed to transfer fiexed sized chunks of bytes know as words.
The number of bytes in a word is a fundamental system parameter that varies across systems. Most have word sizes of 8 bytes
(64 bits)today

- I/O Devices: I/O devices are the system's connection to the external wrold. Each I/O device is connected to the I/O bus
by either a _controller_ or an _adapter_. The distinction between the two is mainly one of packaging. Controllers are chip
sets in the devices itself or on the system's main printed circuit board(_motherboard_). An adapter is a card that plugs
into a slot on the motherboard. Regardless, the purpose of each is to transfer information back and forth between the I/O
bus and an I/O device.

- Main Memory: The main memory is a temporary storage device that holds both a program and the data it mainipulates while
the processor is executing the program. Physically, main memory consists of a collection of _dynamic random access memory_
(DRAM) chips. Logically, memory is organized as a linear array of bytes, each with its own unique address(array index) 
starting at zero.

- Cache: In a typical system, An L1 cache on the processor chip holds tens of thousands of bytes and can be accessed
nearly as fast as the register file. A larger L2 cache with hundreds of thousands to millions of bytes is connected to 
the processor by a special bus. It might take 5 times longer for the process to access the L2 cache than L1 cache. The
L1 cache and L2 cache are implemented with a hardware technology known as _static random access memory_(SRAM).

- Processor: The _central processing unit_(CPU), or simply processor, is the engine that interprets instructions stored
in main memory. At its core is a word-sized storage device(or register) called the _program counter_(PC). At any point 
in time, the PC points at some machine-language instruction in main memory. A processor appears to operate according to
a very simple instruction execution model, defined by its _instruction set architecture_. There are only a few of these
simple operations, and they revolve around memory, the _register file_, and the _arithmetic/logic unit_(ALU). The register
file is a small storage device that consists of a collection of world-sized registers, each with its own unique name. The
ALU computes new data and address values, here are some operations the CPU might carry out:
    - _Load_: Copy a byte or a word from main memory into a register, overwriting the previous contents of the register.
    - _Store_: Copy a byte or a word from a register to a location in main memory, overwriting the previous contents of 
      that location.
    - _Operate_: Copy the contents of two register to the ALU, perform an arthmetic operation on the two words, and store
      the result in a register, overwriting the previous contents of that register.
    - _Jump_: Extract a word from the instruction itself and copy that word into the PC, overwriting the previous value of
      the PC.
  
    
## Operating System ##
The operating system has two primary purposes:
- to protect the hardware from misuse by runaway applications, and
- to provide applications with simple and uniform mechanisms for mainpulating complicated and often wildly different 
low-level hardware devices.

The operating system achieves both goals via the fundametal abstractions: processes, virtual memory, and files. Files
are abstractions for I/O devices, virtual memory is an abstraction for both the main memory and disk I/O devices, and 
processes are abstractions for the processor, main memory and I/O devices.

- Processes: A process is the operating system's abstraction for a running program. By _concurrently_, we mean that the
instructions of one process are interleaved with the instructions of another process. A single CPU can appear to execute
multiple processes concurrently by having the processor switch among them. The operation system performs this interleaving
with a mechanism known as _context switching_. The operating system keeps track of all the state information that the 
process needs in order to run. This state, which is known as _context_, includes information such as the current value of
the PC, the register file, and the contents of main memory. When the operating system decides to transfer control from
the current process to some new process, it performs a context switch by saving the context of the current process, 
restoring the context of the new process, and then passing control to some new process. In modern systems a process can
actually consist of multiple execution units, called _threads_, each running in the context of the process and sharing 
the same code and global data. It is easier to share data between multiple threads that between multiple processes, and
threads are typically more efficient than processes. 

- Virtual Memory: Virtual memory is an abstraction that provides each process with the illsion that it has exclusive use
of the main memory. Each process has the same uniform view of memory, which known as its _virtual address space_. In Linux,
the topmost region of the address space is reserved for code and data in the operating system that is common to all processes.
The low region of the address space holds the code and data defined by the user's process. Starting with the lowest address
to the highest address are: Program code and data, heap, shared libraries, stack, kernel virtual memory.

- Files: __A file is a sequence of bytes__, nothing more or nothing less. Every I/O device, including disks, keyboards, 
display, and even networks, is modeled as a file. All input and output in the system is performed by reading and writing
files.

- Virtual Machine: Providing an abstraction of the entire computer, including the operating system, the process, and the
programs

## Cocurrency and Parallelism ##
The term _concurrency_ refer to the general concept of a system with multiple, simultaneous activities, and the term
_parallelism_ refer to the use of concurrency to make a system run faster. Here are three levels, working from the highest
to the lowest level in the system hierarchy
- Thread-Level Concurrency: With threads, we can even have multiple control flows executing within a single process.
This form of concurrency allows multiple users to interact with a system at the same time, such as many people want to 
get pages from a single Web server. Hyperthreading, sometimes called _simultaneous multi-threading_, is a technique that
allows a single CPU execute multiple flows of control.
- Instruction-Level Parallelism: At a much lower level of abstraction, modern processors can execute multiple instructions
at one time. Processors that can sustain execution rates faster than one instruction per cycle are known as _superscala_
processors.
- Single-Instruction, Multiple Data(SIMD) Parallelism: At the low level, many modern processors have special hardware 
that allows a single instruction to cause  multiple operatios to be performed in parallel. SIMD instructions are provided
mostly to speed up applications that process image, sound and video data.
