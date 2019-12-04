---
title: "computer system 2"
date: 2018-05-28T17:20:36
categories:
- notes
tags:
- system
---

## Assembler ##
Computer execute machine code, sequences of bytes encoding the low-level operations that manipulate data manage memory,
read and write data on storage devices, and communicate over networks.

<!-- more -->

We will focus on x86-64, the commonest machine language used in processor with laptop and PC, also it's widely used in 
supercomputer and lager data center.

the x86-64 machine code is much different with the corresponding C code, the processor states below are everywhere:
- _program counter_: called "%rip", indicates the address in memory of the next instruction to be executed.
- the integer register file contains 16 named location storing 64-bit values.
- the condition code register hold status information about the most recently executed arithmetic or logical instruction.
- a set of vector registers store one or more integer and floating-point data.
for example, the below C code:
```C
//mstore.c
long mult2(long, long);

void multstore(long x, long y, long *dest) {
    long t = mult2(x, y);
    *dest = t;
}
```
could have the assembly code used by `gcc -Og -S mstore.c` command:
```
//mstore.s
	.file	"mstore.c"
	.text
	.globl	multstore
	.type	multstore, @function
multstore:
.LFB0:
	.cfi_startproc
	pushq	%rbx
	.cfi_def_cfa_offset 16
	.cfi_offset 3, -16
	movq	%rdx, %rbx
	call	mult2
	movq	%rax, (%rbx)
	popq	%rbx
	.cfi_def_cfa_offset 8
	ret
	.cfi_endproc
.LFE0:
	.size	multstore, .-multstore
	.ident	"GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.9) 5.4.0 20160609"
	.section	.note.GNU-stack,"",@progbits
```
where the lines beginning with ‘.’ are directives to guide the assembler and linker, we can generally ignore these, thus
we obtain(in ATT format distinguish with Intel format):
```
multstore:
	pushq	%rbx
	movq	%rdx, %rbx
	call	mult2
	movq	%rax, (%rbx)
	popq	%rbx
	ret
```

Intel use term _word_ to represent 16-bit data types, so it calls 32-bit as _double words_, and 64-bit as _quad words_.
The table below illsulates the default C data types and its x86-64 representation.

| C declartive | Intel data type | assembly suffix | size(byte) |
| :----------: | :-------------: | :-------------: | :--------: |
| char | byte | b | 1 |
| short | word | w | 2 |
| int | double words | l | 4 |
| long | quad words | q | 8 |
| char* | quad words | q | 8 |
| float | single-precision | s | 4 |
| double | double-precision | l | 8 |

A x86-64 CPU contains a set of 16 universal destination registers storing 64-bit values. They are: %rax, %rbx, %rcx, %rdx,
%rsi, %rdi, %rbp, %rsp(in 32-bit those begin with e, e.g. %eax), %r8 to %r15(in 32-bit those ends with d, e.g. %r9d). 
It's worth noting that the stack point %rsp, which indicates the address of program finished stack.

### data movement instructions ###
Among the most heavily used instructions are those that copy data from one location to another. The most simplest instructions
is the MOV class, cosists of four instructions: `movb`, `movw`, `movl`, `movq`; they differ only in that they operate on data 
of size 1, 2, 4 and 8 bytes. In most casees, MOV only update the register and memory address indicated by the operands,
but `movl` will fill the left bit with 0 in a register with a 32-bit data, which is the convention in x86-64. The MOVZ class
will always fill the other bit with zero, while the MOVS class fill by sign expansion. MOVS and MOVZ always ends with 
two characters indicates the size of size, e.g. `movzbw` moves byte to word with zero expansion. but there no movzlq
because it could replaced by `movl` as we mentioned above. The `cltq` always moves from %eax to %rax with sign expansion,
so it is the same as `movslq %eax, %rax`.

The final two data movement instructions are used to push data onto and pop data from the program stack: pushq and popq.
For example to push a quad words onto a stack, first we subtract %rsp(the stack point) with 8(the stack grows from high
address to low address) and write the data to the new address of %rsp. Since the stack is contained in the same memory as
the program code and other forms of program data, programs can access arbitrary positions within the stack using the 
standard memory addressing methods.

### arithmetic and logical operations ###

| Instruction | Effect | Description |
| :---------: | :----: | :---------: |
| leal    S, D | D := &S | Load effective address |
| INC    D | D := D + 1 | Increment |
| DEC    D | D := D - 1 | Decrement |
| NEG    D | D := -D | Negate |
| NOT    D | D := ~D | Complement |
| | | |
| ADD    S, D | D := D + S | Add |
| SUB    S, D | D := D - S | Subtract |
| IMUL   S, D | D := D * S | Multiply |
| XOR    S, D | D := D ^ S | Exclusive-or |
| OR     S, D | D := D &#124; S | Or |
| AND    S, D | D := D & S | And |
| | | |
| SAL    k, D | D := D << k | Left shift |
| SHL    k, D | D := D << k | the same as SAL |
| SAR    k, D | D := D >> k | Arithmetic right shift(with sign expansion) |
| SHR    k, D | D := D >> k | Logical right shift(with zero expansion) |

The _load effective address_ instruction leal is actually a variant of the movl instruction. It just works as the `&` in C.
In addition, it can be used co compactly describe common arithmetic operations, e.g. if register %edx contains value x,
then the instruction `leal 7(%edx, &edx, 4), %eax` will set register %eax to 5x+7. But the destination operand must be a
register.

For shift operations, the shift amount is given either as an immediate or in the single-byte register element %c1(in %rcx),

### control ###
 Machine code provides two basic low-level mechanisms for implementing coditional behavior: it tests data values and then
either alters the control flow or the data flow based on the result of these test. Data-dependent control flow is the
more general and more common approach for implementing coditional behavior. The execution order of a set of machine-code
instructions can be altered with _jump_ instruction, indicating that control should pass to some other part of the program,
possibly contingent on the result of some test.

In addition to the integer register, the CPU maintains a set of single-bit _condition code_ register describing attributes
of the most rescent arthmetic or logical operation. These register can then be tested to perform conditional branches. 
The most useful cndition codes are:
- CF: Carry Flag. The most recent operation generated a carry out of the most significant bit. Used to detect overflow
for unsigned operations
- ZF: Zero Flag. The most recent operation yielded zero.
- SF: Sign Flag. The most recent operation yielded negative value.
- OF: Overflow Flag. The most recent operation caused a two's-complement overflow, either negative or positive.

For example, suppose we used one of the ADD instructions to perform the equivalent of the C assignment `t=a+b`, where
variables a, b, and t are integers. Then the condition codes would be set according to the following C expressions:
- CF: `(unsigned) t < (unsigned) a`    Unsigned overflow
- ZF: `t == 0`     Zero
- SF: `t < 0`      Negative
- OF: `(a < 0 == b < 0) && (t < 0 != a < 0)`    Signed overflow

There are two instruction classes that set condition codes without altering any other registers, the CMP class and TEST
class. The CMP instructions behave in the same manner as the SUB instructions except that they don't update their destinations.
The TEST instructions behave in the same manner as the And instructions except that they don't update their destinations,
too.

Rather than reading the condition codes directly, there are three common ways of using the codition code:
1. we can set a single byte to 0 or 1 depending on somecombination of the condition codes
2. we can coditionally jump to some other part og the program, or
3. we can conditionally transfer data.

For the first case, there are SET class instructions to set a single byte to 0 or 1 depending on some combination of the
condition codes. e.g. `setl` and `setb` denote "set less" and "set below"

A _jump_ instruction can cause the execution to switch to a completely new position in the program. These jump destinations
are negerally indicated in assembly code by a _label_


## Translating Conditional Branches ##
The most general way to translate conditional expressions and statement from C into machine code is to use combinations
of conditional and unconditional jumps.

The general form of an if-else statement in C is given by the template
```C
if (test-expr)
    then-statement
else
    else-statement
```
for this general form the assmebly implementation typically adheres to the following form, where we use C syntax(use goto)
to describe the control flow.
```C
t = test-expr
if (!t)
    goto false;
then-statement
goto done;
false:
    else-statement
done:
```

Translating conditional branches by control is simple and in common use. But in modern processor, it sometimes may be 
much inefficient. A substitution strategy is using data to translate conditional branches, this strategy compute both
branches' result and choose one according to the condition. It could only be feasible in some constraint situation, but
if feasible, then we can use one conditional transfer instruction to implement.

To explain why using conditional transfer is faster than using control(jump), we need to know how instructions are executed
in processor. Moder processor gains high performance by using pipelining. In pipelining, it takes many phases to process
a instruction, such like read instruction from memory, determine the class of instruction, etc. This method obtains high
performace by coinciding sequential those phases of instructions, e.g. the processor could execute the arithmetic of the
previous instruction while reading the current instruction. To do this, the processor need to know the order of instructions
sequence before, to ensure that there are enough instruction to execute. Then we stand on the conditional branch, the 
processor need to compute what branch to go. Though moder processor using very accurate branch prediction logic to predict
which jump instructions will be execute(the accuracy > 90%) to assure there are enough instruction in the pipelining, it
have to deprecate the work done for the wrong jump instruction, which takes 15~30 clock cycle. On the other hand, it would
always takes 8 clock cycle by data transfer, it not depending what the data like, thus it would be easy to fill the piplining
with instructions.

The CMOVE class implaments for conditional move(e.g. cmovene S R, move data from S to R while). Consider the expression
`v = test-expr : then-expr : else-expr`, using control to compile it could be below:
```C
if (!test-expr)
    goto false;
v = then-expr;
goto done;
false:
    v = else-expr;
done:
```
either then-expr or else-expr would be execute, while using conditional transfer it would execute both of them:
```C
v = then-expr;
ve = else-expr;
t = test-expr;
if(!t) v = ve;
```
So, it's easy to see that it could be much slow if the cost if `then-expr` and `else-expr` is too expensive.

## Loop ##

### do-while loop ###
do-while have the form below:
```C
do
    body-statement
while (test-expr)
```
it can be translate into conditionals and goto statements as follow:
```C
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
```

### while loop ###
while loop have the form below:
```C
while (test-expr)
    body-statement
```

There multiple ways to translate it, the GCC uses two of them, the first is called _jump to middle_, in goto form that is:
```C
goto test;
loop:
    body-statement
test:
    t = test-expr;
    if (t)
        goto loop;
```
the second method is called guarded-do, it first translate to the do-while form:
```C
t = test-expr;
if (!t)
    goto done;
do
    body-statement
while (test-expr);
done:
```
futher more, it can be translated below:
```C
t = test-expr;
if (!t)
    goto done;
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
done:
```
GCC would translate while loop in guarded-do form with `-O1` option.

### for loop ###
for loop has the form below:
```C
for (init-expr; test-expr; update-expr)
    body-statement
```
C standard indicate that it is the same as the while loop below(except one situation):
```C
init-expr
while (test-expr)
    body-statement
    update-expr;
```
So GCC translate the for loop as same as the while loop above, choosing which form depending to the optimization degree.

### switch statement ###
A switch statement provides a multi way branching capability vased on the value of an integer index. They are particularly
useful when dealing with tests where there can be a large number of possible outcomes. Not only do they make the C code
more readable, they also allow an efficiant implementation using a data structure called _jump table_. JUmp tables are 
used when there are a number of cases(e.g. four or more) and they span a small range of values. For example:
```C
void switch_eg(long x, long n, long *dest) {
    long val = x;
    switch (n) {
        case 100:
            val *= 13;
            break;
        case 102:
            val += 10;
        case 103:
            val += 11;
            break;
        case 104:
        case 106:
            val *= val;
            break;
        default:
            val = 0;
    }
    *dest = val;
}
```
the code above would be translated below by a jump table:
```
switch_eg:
.LFB0:
	.cfi_startproc
	subq	$100, %rsi
	cmpq	$6, %rsi
	ja	.L8
	jmp	*.L4(,%rsi,8)               the jump table
	.section	.rodata
	.align 8
	.align 4
.L4:
	.quad	.L3
	.quad	.L8
	.quad	.L5
	.quad	.L6
	.quad	.L7
	.quad	.L8
	.quad	.L7
	.text
.L3:
	leaq	(%rdi,%rdi,2), %rax
	leaq	(%rdi,%rax,4), %rdi
	jmp	.L2
.L5:
	addq	$10, %rdi
.L6:
	addq	$11, %rdi
	jmp	.L2
.L7:
	imulq	%rdi, %rdi
	jmp	.L2
.L8:
	movl	$0, %edi
.L2:
	movq	%rdi, (%rdx)
	ret
	.cfi_endproc
```

## Procedures ##
A procedure call involves passing both data and control form one part of program to another, it could have different form
in different language: function, method, subroutine, handler. To supprot procedures, machine must have the abilities:
- transfer control
- transfer data
- allocate and deallocate memory sapce.

### runtime stack ###
While the x86-64 procedure need more spaces than register have, then it would allocate space on stack, which called a
stack frame of the procedure. If all local variables could be stored in register, the procedure don't need the stack frame.

### transfer control ###
To transfer control from procedure P to Q, it just simply set the PC with the start of Q, x86-64 uses `call Q` to push 
address A and then set the PC with the start of Q, the corresponding instruction `ret` would pop the address A, and set
the PC with A.
### transfer data ###
In x86-64, it can store 6 parameters in register at most, for 64-bit size data, they stored in the order: %rdi, %rsi, %rdx
%rcx, %r8, %r9. So the parameters beyond should transfer by stack.

### register local space ###
Register is the only resource shared by all procedures. Conventionally, register %rbx, %rbp and %r12 ~ %r15 are called
_callee-saved_ registers, this means when procedure Q is called by P, then Q must save the values of any of these registers
on the stack before overwriting them and restore them before returning. Any other registers except the stack point %rsp 
are called _caller-saved_ registers, this means any procedure can overwriting this registers without restoring. The prefix
of "saved" means who should store the data before pass the control to another procedure.

## Out-of-Bounds Memory References and Buffer Overflow ##
We have see that C does not perform any bounds checking for array references, and that local variables are stored on the
stack along with state information such as saved register values and return addresses. This combination can lead to serious
program errors, where the state stored on the stack gets corrupted by a write to an out-of-bounds array element. When the
program then tries to reload the register or execute a `ret` instruction with this corrupted state, things can go seriously
wrong.
