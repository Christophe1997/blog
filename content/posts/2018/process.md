---
title: 进程
date: 2018-08-03 13:10:58
categories:
- Things I Learned
tags:
- System
---

The abstraction provided by the OS of a running program is something we call a process. And there are some APIs must be
included in any interface of an operating system:
<!-- more -->
1. create
2. destroy
3. wait
4. miscellaneous control, most OS provide some kind of method to suspend a process and resume it.
5. status, there are usually interfaces to get some status information about a process.

The first thing that the OS must do to run a program is to load its code and any static data into memory, into the address
space of the process. Once the code and static data are loaded into memory, there are a few other things the OS needs to
do before running the process. Some memory must be allocated for the program's _run-time stack_, and the OS may also 
allocate smoe memory for the program's heap. Also, the os will do some other intialization tasks, particularly as related
to I/O, i.e. in Unix systems each process by default has three open file descriptors, for standard input, output and error.

At any time a process may be _running_, _ready_ or _blocked_. Being moved from ready to running means the process has been
_scheduled_ and _descheduled_ in contrast. Once a process has become blocked, the OS will keep it as such until some event
occurs; at that point, the process moves to the ready state again.

The OS is a program, and like any program, it has some key data structures that track various relevant pieces of information.
For example, the tracked information about each process in xv6 kernel is shown below:
```C
// the registers xv6 will save and restore to stop and subsequently restart a process
struct context {
    int eip;
    int esp;
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;
};

// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// the information xv6 tracks about each process including its register context and state
struct proc {
    char *mem;                   // Start of process memory
    uint sz;                     // Size of process memory
    char *kstack;                // Bottom of kernel stack for this process
    enum proc_state state;       // Process state
    int pid;                     // Process ID
    struct proc *parent;         // Parent process
    void *chan;                  // If non-zero, sleeping on chan
    int killed;                  // If non-zero, have been killed
    struct file *ofile[NOFILE];  // Open files
    struct inode *cwd;           // Current directory
    struct context context;      // Switch here to run process
    struct trapframe *tf;        // Trap frame for the current interrupt
};
```
## Process API ##
Unix presents one of the most intriguing ways to create a new process with a pair of system calls: `fork()` and `exec()`.
`wait()` can be used by a process wishing to wait for a process it has created to complete.

The `fork()` creates a process as an(almost) _exact copy of the calling process_, that means that to the OS, it now looks
like there are two copies of the program(the caller) running, and both are about to return from the `fork()` system call.
The newly-created process doesn't run the `main()` again, instead it just comes into life as if it had called `fork()` 
itself. To distinguish parent and child, the `fork()` return non-negative PID in parent process, and return 0 in child
process, and if it return a negative number, that means it failed. Now there are now two active processes in the system 
after calling fork, and it's non-determinism to know which runs first. 

Sometimes, it's useful for a parent to wait a child process to finish what it has been doing, this is accomplished with
the `wait()` system call. the process calls `wait()` to delay its execution, most until the child has run and exited. When
`wait()` return into parent, returns the PID of child.

By giving a name of an executable to `exec()`(i.e. `execvp`), it loads code from that executable and overwrites its current
code segment with it. The heap and stack and other parts of the memory space of the program are re-initialized. Then the
OS simply runs that program, passing in any arguments as the `argv` of that process. Thus it does not create a new process,
rather, it transforms the currently running program into a different running program. A successful call to `exec` never
returns.

The spearation of `fork()` and `exec()` is essential in building a Unix shell, because it lets the shell run code after
the call to `fork()` but before the call to `exec()`. For example, shell can redirect the output from stdout to a new file.
The way the shell accomplishes this task is quite simple: when the child is created, before calling `exec()`, the shell
closes standard output and open the file, thus any output from the soon-to-be-running program are sent to the file instead
of the screen. This works because Unix system start looking for free file descriptors at zero, in this case, `STDOUT_FILENO`
will be the first available one and thus get assigned when open() is called. Subsequent writes by the child process to 
the standard output file descriptor(i.e. printf) will then be routed transparently to the newly-opened file. 

## Limited Direct Execution ##
In order to virtualize the CPU, the operating system needs to somehow share the physical CPU among many jobs running 
seemingly at the same time. The basic idea is simple: run one process for a little while, then run another one, and so
forth.

To make a program run as fast as one might expect, not surprisingly OS developers came up with a technique, which we 
call _limited direct execution_. The direct execution means run the program directly on the CPU, but only "direct execution"
is not enough, the OS need to prevent what we don't want it to do, thus we need "limited".

There are two phases in the limited direct execution protocol. In the first(at boot time), the kernel initializes the 
trap table, and the CPU remembers its location for subsequent use. The kernel does so via a privileged instruction. In
the second(when running a process), the kernel sets up a few things before using a return-from-trap instruction to start
the execution of the process; this switches the CPU to user mode and begins running the process. When the process wishes
to issue a system call, it traps back into the OS, which handles it and once again returns control via a return-from-trap
to the process. The process then completes its work, and returns from `main()`; this usually will return into some stub
code which will properly exit the program. At this point, the OS cleans up and we are done.

If a process is running on the CPU, this means the OS is not running. So there is a problem that how can OS regain control
of the CPU so that it can switch between processes. One old way is waiting system call but the OS can not do much at all if
the process refuses to make system call. Another way is to add a _timer interrupt_, a timer device can be programmed to
raise an interrupt every so many milliseconds; when the interrupt is raised, the currently running process is halted, and
a pre-configured interrupt handler in the OS runs. At this point, the OS has regained control of the CPU.

## Scheduling Policies ##
There are many scheduling algorithms, such as FIFO(first in first out), STCF(shortest time-to-completion first). Choosing
which algorithm dependent on which metrics are used. One most well-known approaches to scheduling, known as the _Multi-level
 feedback queue_. The MLFQ has a number of distinct queues, each assigned a different priority level. At any given time,
a job that is ready to run is on a single queue. And a job with higher priority is chosen to run first. If two jobs have
the same priority, we can use round-robin scheduling. The key to MLFQ scheduling therefore lies in how scheduler sets
priorities. Rather than giving a fixed priority to each job, MLFQ varies the priority of a job based on its observed 
behavior. If, for example, a job repeatedly relinquishes the CPU while waiting for input from the keyboard, MLFQ will
keep its priority high, as this is how an interactive process might behave. 

MLFQ first assumes the arrived job as a short job, thus giving the job high priority. If it actually is a short job, then
it runs quickly and complete, if not, MLFQ slowly move down its priority. And a simple MLFQ rules may be:
1. If Priority(A) > Priority(B), A runs.
2. If Priority(A) = Priority(B), A&B run in round-robin.
3. when a job enters the system, it is placed at the highest priority.
4. Once a job uses up its time allotment at a given level, its priority is reduced.
6. after some time period S, move all the jobs in the system to the topmost queue.

Another scheduling policy is _lottery scheduling_ as a proporitional-share scheduling, the basic idea is simple: every so
often, hold a lottery to determine which process should get to run next; process that should run more often should be
given more chances to win the lottery(to be continued).
