---
title: "Computer System 3"
date: 2018-06-01T23:23:16
categories:
- Notes
tags:
- System
---

## Optimization ##

Writing an efficient program requires several type of activities. First, we must select an appropriate set of algorithms
and data structures. Second, we must write source code that the compiler can effectively optimize to turn into efficient
executable code. A third technique for dealing with especially demanding computations is to divide a task into portions
that can be computed in parallel, on some combination of multiple cores and multiple processors.
<!-- more -->

And our goal in optimization is to keep modifying the source code in attempt to coax the compiler into generating efficient
code. Compared to the alternative of writing code in assembly language, this indirect approach has the advantage that
the resulting code will still run on other machines, although perhaps not with peak performance.

Compilers must be careful to apply only safe optimizations to a program, meaning that the the resulting program will have
the exact same behavior as would an unoptimized version for all possible cases the program may encounter.

The first problem is the _memory aliasing_ that two pointers designate the same memory location, and in performing only 
safe optimizations, the compiler must assume that different pointers may be aliased. If a compiler cannot determine whether
or not twopointers may be aliased, it must assume that either case is possible, limiting the set of possible optimizations

A second optimization blocker is due to function calls. It's hard to optimize the function calls with side effect, which
modifies some part of the global program state. Most compilers do not try to determine whether a function is free of side
effects, instead the compiler assumes the worse case and leaves function calls intact. Among compilers, GCC is considered
adequate, but not exceptional.

We introduce the metric _cycles per element(CPE)_, CPE measurements help us understand the loop performance of an iterative
program at a detailed level. The sequencing of activities by a processor is controlled by a clock providing a regular
signal of some frequency, usually expressed in _gigahertz(GHz)_, billions of cycles per second. For example, when product
literature characterizes a system as "4 GHz" processor, it means that the processor clock runs at 4.0E9 cycles per second,
and the period of the processor is 0.25 nanoseconds, which is the reciprocal of the clock frequency.

For example, consider the two programs of computing the prefix sum of a vector a:

```C
void psum1(float a[], float p[], long n) {
    long i;
    p[0] = a[0];
    for (i = 1; i < n; i++) {
        p[i] = p[i - 1] + a[i];
    }
}

void psum2(float a[], float p[], long n) {
    long i; 
    p[0] = a[0];
    for (i = 1; i < n - 1; i += 2) {
        float mid_val = p[i - 1] + a[i];
        p[i] = mid_val;
        p[i + 1] = mid_val + a[i + 1];
    }
    if (i < n) {
        p[i] = p[i - 1] + a[i];
    }
}
```

They takes 368 + 9.0n cycles and 368 + 6.0 cycles, for lagre N the run times will be dominated by the linear factors. We
refer to the coefficients in these terms as the effective number of cycle per element, so the first one's CPE is 9.0 and
the second one is 6.0.

### Modern processors ###
One of the remarkable feats of modern microprocessors is they employ complex and exotic microarchitectures, in which 
multiple instructions can be executed in parallel, while presenting an operational view of simple sequential instruction
execution. There are two different lower bounds characterize the maximum performance of a program. The _latency bound_
is encountered when a series of operations must performed in strict sequence, because the result of one operations is 
required before the next one can begin. This bound can limit program performance when the data dependencies in the code
limit the ability of the processor to exploit instruction-level parallelism. The _throughput bound_ characterizes the
raw computing capacity of the processor's functional units. This bound becomes the ultimate limit on program performance.

### Trick ###

- _code motion_: Identify a computation that is performed multiple times but such that the result of the computation will
not change, and therefor move the computation to an eailer section of the code that does not get evaluated as often.
- _eliminate unneeded menory reference_: For frequent memory reference, it could be improved by introduced a temporary
variables instead of using memory referencing(pointer) in a loop.
- _loop unrolling_: Reduce the number of iterations for a loop by increasing the number of elements computed on each 
iteration. It can improve performance in two ways. First, it reduces the number of operations that do not contribute
directly to the program result, such as loop indexing and conditional branching. Second, it expose ways in which we can 
futher transform the code to reduce the number of operations in the cirtical paths of the overall computation.

## The Memory Hierarchy ##

### Random-access memory ###
_Random-access memory_(RAM) comes in two varieties, static and dynamic. _Static RAM_(SRAM) is faster and significantly 
more expensive than _dynamic RAM_(DRAM). SRAM is used for cache memories, both on and off the CPU chip. DRAM is used for
the amin memory plus the frame buffer of a graphics system.

### Read-only memory ###
_Read-only memory_ can retain their information even when they are powered off, while the RAM lose their information if
the supply voltage is turned off. _flash memory_ is a type of nonvolstile memory, based on EEPROM(_electrically erasable
 programmable ROM_), that has become an important storage rechnology. The _solid state disk_(SSD), that provides a faster,
 sturdier and less power-hungry alternative to conventional rotating disk is a new form of flash-based disk driver.

 ## Principle of Locality ##
 programs with good locality run faster than programs with poor locality. Locality is typically described as having two
 distinct forms: _temporal locality_ and _spatial locality_. In a program with good temporal locality, a memory location
 that is referenced once is likely to be referenced again multiple times in the near future. In a program with good spatial
 locality, if a memory location that is referenced once, then the program is likely to reference a nearby memory location
 in the near future.

Consider a program of matrix multiplication(C=AxB):
```C
for (i = 0; i < n; i++)
    for (j = 0; j < m; j++) {
        sum = 0.0;
        for (k = 0; k < p; k++) {
            sum += A[i][k] * B[k][j];
        }
        C[i][j] = sum;
    }
```
if we switch the loop order for j and k:
```C
for (i = 0; i < n; i++)
    for (k = 0; k < p; j++) {
        r = A[i][k]
        for (j = 0; j < m; k++) {
            C[i][j] += r * B[k][j];
        }
    }
```
these will not change the result, but the second improves efficiency. Because the second has no cache missing.

- Programs that repeatedly reference the same variables enjoy good temporal locality.
- For programs with stride-k reference patterns, the smaller the stride the better the spatial locality. Programs with
stride-1 reference patterns have good spatial locality. Programs that hop around memory with large strides have poor
spatial locality.
- Loops have good temporal and spatial locality with respect to instruction fetches. The smaller the loop body and the 
greater the number of loopiterations, the better the locality.
