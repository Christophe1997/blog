---
title: "Computer System 4"
date: 2018-06-10 19:14:22
categories: 
- Things I Learned 
tags:
- System
---

## Linking ##
Linking is the process of collecting and combining various pieces of code and data into a single file that can be loaded
(copied) into memory and executed. Linking can be performed at compile time, when the source code is translated into 
machine code; at load time, when the program is loaded into memory and executed by the _loader_; and even at run time, by
application programs. On modern systems, linking is performed automatically by programs called _linkers_.
<!-- more -->

### Static linking ###
_Static linker_ such as the Unix ld program take as input a collection of relocatable object files and command-line arguments
and generate as output a fully linked executable object file that can be loaded and run. To build the executable, the
linker must perform two main tasks:
1. _Symbol resolution_. Object files define and reference symbols. The purpose of symbol resolution is to associate each
symbol reference with exactly one symbol definition.
2. _Relocation_. Compilers and assemblers generate code and data sections that start at address 0. The linker _relocates_
these sections by associating a memory location with each symbol definition, and then modifying all of the references
to those symbols so that they point to this memory location.

### Object file ###
Object files are merely collections of blocks of bytes, it comes in three forms:
- _Relocatable object file_. Contains binary code and data in a form that can be combined with other relocatable object
files at compile time to create an executable object file.
- _Executable object file_. Contains binary code and datain a form that can be copied directly into momory and executed.
- _Shared object file_. A special type of relocatable object file that can be loaded into memory and linked dynamically,
at either load time or run time.

Modern x86-64 Linux and Unix system use _Executable and Linkable Formate_(ELF) to format object file, A typical ELF 
relocatable object file contains the following sections:
- .text: The machine code of the compiled program.
- .rodata: Read-only data such as the format strings in printf statements, and jump tables for switch statements.
- .data: Initialized global C variables.
- .bss: Uninitialized global C variables. This section occupies no actual space in the object file; it is merely a place 
holder.
- .symtab: A symbol table with information about functions and global variables that are defined and referenced in the 
program.
- .rel.text: A list of locations in the .text section that will need to be modified when the linker combines this object
file with others.
- .rel.data: Relocation information for any global variables that are referenced or defined by the module.
- .debug: A debugging symbol table with entries for local variables and typedefs defined in the program, global variables
defined and referenced in the program, and the original C source file.
- .line: A mapping between line numbers in the original C source program and machine code instructions in the .text section.
- .strtab: A string table for the symbol tables in the .symtab and .debug sections, and for the section names in the 
section headers.

The .debug and .line sections only present if the compiler driver is invoked with the -g option.

### Symbol ###
Each relocatable object module m, has a symbol table that contains information about the symbols that are defined and 
referenced by m. In the context of a linker, there are three different kinds of symbols:
- _Global symbols_ that are defined by module m and that can be referenced by other modules. Global linker symbols 
correspond to _nonstatic_ C functions and global variables that are defined without the C `static` attribute.
- _externals_ that are the Global symbols referenced by module m but defined by some other module.
- _Local symbols_ that are defined and referenced exclusively by module m. Some local linker symbols correspond to C 
functions and global variables that are defined with the `static` attribute. These symbols are visible anywhere within 
module m, but cannot be referenced by other modules.

Interestingly, local procedure variables that are defined with the C static attribute are not managed on the stack. 
Instead, the compiler allocates space in .data or .bss for each definition and creates a local linker symbol in the 
symbol table with a unique name.

Symbol tables are built by assemblers, using symbols exported by the compiler into the assembly-language .s file, it 
contains an array of entries, each entry has the format below:
```C
typedef struct {
int name;           /* String table offset */
char type:4,        /* Function or data (4 bits) */
     binding:4;     /* Local or global (4 bits) */
char reserved;      /* Unused */
short section;      /* Section header index */
long value;          /* Section offset, or absolute address */
long size;           /* Object size in bytes */
} Elf64_Symbol;
```

The linker resolves symbol references by associating each reference with exactly one symbol definition from the symbol 
tables of its input relocatable object files. It's much easy to reference local symbols than global symbols.The compiler
allows only one definition of each local symbol per module. Overloaded functions in C++ and Java work because the compiler 
encodes each unique method and parameter list combination into a unique name for the linker, which is known as mangling.

At compile time, the compiler exports each global symbol to the assembler as either _strong_ or _weak_, and the assembler
encodes this information implicitly in the symbol table of the relocatable object file. Functions and initialized global
variables get strong symbols. Uninitialized global variables get weak symbols. Given this notion of string and weak symbols,
Unix linkers use the following rules for dealing with multiply defined:
1. Multiple strong symbols are not allowed.
2. Given a strong symbol and multiple weak symbols, choose the strong symbol.
3. Given multiple weak symbols, choose any of the weak symbols.

## Exceptional Control Flow ##
Exceptions can be divided into four classes: _interrupts_, _traps_, _faults_ and _aborts_.

_Interrupts_ occur asynchronously as a result of signals from I/O devices that are external to the processor. Exception
handlers for hardware interrupts are often called _interrupt handlers_. After the current instruction finishes executing,
the processor notices that the interrupt pin has gone high, reads the exception number from the system bus, and then calls
the appropriate interrupt handler. When the handler return, it returns control to next instruction. The remaining classes
of exceptions(traps, falusts and aborts) occur synchronously as a result of executing the current instruction. We refer
to this instruction as the _faluting instruction_.

_Traps_ are intentional exceptions that occur as a result of executing an instruction. The most import use of traps is 
to provide a procedure-like interface between user programs and the kernel known as _system call_. From a programmer's
perspective, a system call is identical to a regular function call. However, regular functions run in user mode, which 
restricts the types of instructions they can execute, and they acess the same stack as the calling function, while a 
system call runs in kernel mode, which allows it to execute instructions, and accesses a stack defined in the kernel.

_Faults_ result from error conditions that a handler might be able to correct. When a fault occurs, the processor 
transfers control to the fault handler. If the handler is able to correct the error condition, it returns control to the
faulting instruction, thereby reexecuting it. Otherwise, the handler returns to an abort routine in the kernel that 
terminates the application program that caused the fault.

_Aborts_ result from unrecoverable faltal errors, typically hardware errors.

x86-64 system has up to 256 different exception types. Numbers in the range from 0 to 31 corresnpond to execeptions that
are defined by Intel, while numbers in the range from 32 to 255 correspond to interrupts and traps that are defined by 
the operating system.

### Processes ###
Exceptions are the basic building blocks that allow the operating system to provide the notion of a _process_, one of 
the most profound and successful ideas in computer science. The classic definition of a process is an _instance of a 
program in execution_. A process provides two key abstractions: An independent logical control flow and A private address
space.

Unix provides a number of system calls for manipulating processes from C programs. Each process has a unique positive 
(nonzero) process ID (PID). The getpid function returns the PID of the calling process. The getppid function returns the
PID of its parent. The _fork_ function creates a new running child process. The fork function is called once but returns
twice: once in the calling process (the parent), and once in the newly created child process. In the parent, fork returns
the PID of the child. In the child, fork returns a value of 0. For example:
```C
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main () {
    pid_t pid;
    int x = 1;

    pid = fork();
    if (pid == 0) {
        printf("child : x=%d\n", ++x);
        exit(0);
    }

    printf("parent: x=%d\n", --x);
    exit(0);
}
```
And it's result in Unix system is
```
parent: x=0
child: x=2
```
Here you can see the fork function call once, return twice. And the parent and the child are sperate processes that run
concurrently, we can never make assumptions about the order of the two process. Those two process also share duplicate but
separate address spaces, they both have x equals to 1, but they do different things to x sperately. Also they share files,
that means the child process inherits all of parent's open files. 

### Common memory-related bugs in C ###

1. Dereference bad pointers: If we attempt to dereference a pointer into the unmapped virtual address or the read-only
areas and write, the operating system will terminate the program with a segmentation exception. But if the address is
legal, then it will never report a problem, which usually cause baffling consequences.

2. Read uninitialized memory: While bss memory locations(such as unintialized global C variables) are always initialized
to zero by the loader, this is not true for heap memory. You should always zero it explicitly or use calloc.

3. Allow stack buffer overflows: A program has a buffer overflow bug if it writes to a target buffer on the stack without
examining the size of the input string.

4. Assume that pointers and the objects they point to are the same size, It is always incorrect.

5. Make off-by-one errors: index out of range.

6. Reference a pointer instead of the object it points to.

7. Misunderstande pointer arithmetic.

8. Reference nonexistent variables: always means return a pointer point to the local variable.

9. Reference data in free heap blocks: reference the data after it frees.

10. Introduce memory leaks: memory leaks are particularly serious for programs such as daemons and servers, which by 
definition never terminate.

## System-level I/O ##
A Unix file is a sequence of m bytes:
$$ B_0, B_1, \cdots, B_k, \cdots, B_{m-1} $$
All I/O devices, such as networks, disks, and terminals, are modeled as files, and all input and output is performed by
reading and writing the appropriate files.This elegant mapping of devices to files allows the Unix kernel to export a 
simple, lowlevel application interface, known as Unix I/O, that enables all input and output to be performed in a uniform
and consistent way:
1. Opening files.
2. Changing the current file position.
3. Reading and writing files.
4. Closing files.

Every Unix file has a type to show their role in system:
1. _regular file_ contains any data, applications always need to distinguish between text file and binary file.
2. _directory_ contains a set of links. Each link maps a filename to a file. "." links the directory itself, and ".."
links the parent directory.
3. _socket_ is used for cross-network communication.
and so on(named pipe, symbolic link). Every process have a _current working directory_ in its context to refer the work
location.
