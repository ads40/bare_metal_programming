+++
title = "Set up for learning Armv8-A assembly"
date = 2023-10-01
draft = true

[taxonomies]
categories = ["arm-asm"]
tags = ["linux", "rpi4"]

[extra]
lang = "en"
toc = true
comment = false
copy = true
math = false
mermaid = false
outdate_alert = false
outdate_alert_days = 120
display_tags = true
truncate_summary = false
+++

One approach to learning about Armv8-A assembly is simply to do the learning
directly on a Raspberry Pi with the standard `gcc` toolchains that are already
in place. This gives direct access to an Armv8-A (assuming that you are running
a 64-bit operating system on the Pi) processor, and you can experiment directly
on the machine.

See also this post for an alternative approach using QEMU "user mode":

- [Set up for learning Armv8-A assembly via QEMU User Mode](/posts/set-up-for-armv8-asm-qemu-user/)

## Set up

Using a Raspberry Pi 4B as a native `arm64` target requires no special set-up
since this is the machine's native architecture.

```sh
$ uname -m
aarch64
$ lscpu
Architecture:                    aarch64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       1
Vendor ID:                       ARM
Model:                           3
Model name:                      Cortex-A72
Stepping:                        r0p3
CPU max MHz:                     1800.0000
CPU min MHz:                     600.0000
BogoMIPS:                        108.00
<...output truncated...>
Flags:                           fp asimd evtstrm crc32 cpuid
```

Create a test file`adder.c` in C.

```c
/* adder.c */
#include <stdio.h>

int addTwo(int a)
{
    return a + 2;
}

int main()
{
    int x = 40;
    printf("Value of x is:         %d\n", x);
    x = addTwo(x);
    printf("Value of addTwo(x) is: %d\n", x);
    return 0;
}
```

Compile `adder.c`, and then dump the assembly to see the Armv8-A assembly
produced by the compiler for `adder.c`:

```sh
$ gcc -std=c99 -Wall -g -o build/adder adder.c -lm
$ objdump -d build/adder > build/adder.outasm

# Lines 139-168 of adder.outasm are assembly versions of the C code above.
#
#  0000000000000774 <addTwo>:
#   774:	d10043ff 	sub	sp, sp, #0x10
#   778:	b9000fe0 	str	w0, [sp, #12]
#   77c:	b9400fe0 	ldr	w0, [sp, #12]
#   780:	11000800 	add	w0, w0, #0x2
#   784:	910043ff 	add	sp, sp, #0x10
#   788:	d65f03c0 	ret
#
# 000000000000078c <main>:
#   78c:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
#   790:	910003fd 	mov	x29, sp
#   794:	52800500 	mov	w0, #0x28                  	// #40
#   798:	b9001fe0 	str	w0, [sp, #28]
#   79c:	b9401fe1 	ldr	w1, [sp, #28]
#   7a0:	90000000 	adrp	x0, 0 <_init-0x5d0>
#   7a4:	91220000 	add	x0, x0, #0x880
#   7a8:	97ffffaa 	bl	650 <printf@plt>
#   7ac:	b9401fe0 	ldr	w0, [sp, #28]
#   7b0:	97fffff1 	bl	774 <addTwo>
#   7b4:	b9001fe0 	str	w0, [sp, #28]
#   7b8:	b9401fe1 	ldr	w1, [sp, #28]
#   7bc:	90000000 	adrp	x0, 0 <_init-0x5d0>
#   7c0:	91228000 	add	x0, x0, #0x8a0
#   7c4:	97ffffa3 	bl	650 <printf@plt>
#   7c8:	52800000 	mov	w0, #0x0                   	// #0
#   7cc:	a8c27bfd 	ldp	x29, x30, [sp], #32
#   7d0:	d65f03c0 	ret
#   7d4:	d503201f 	nop
#   7d8:	d503201f 	nop
#   7dc:	d503201f 	nop
```

## Building binaries from assembly code and starting the debugger

Working directly with hand-written assembly involves an extra step or two, and
it is a good idea to be able to use the debugger to step through the compiled
code to see how the assembly instructions play out at the register level.

Compilation from assembly requires a two-step process of first assembling and
then linking the object file into an executable. These steps are also carried
out when compiling C code, but the `gcc` executable takes charge of the whole
compiler pipeline.

Create a test program in Armv8-A assembly.

```c
/* hello_world.s */
.section .text
.global _start

_start:
/* Linux syscall write(int fd, const void *buf, size_t count) */
    mov x0, #1     
    ldr x1, =msg 
    ldr x2, =len 
    mov w8, #64 
    svc #0

/* Linux syscall exit(int status) */
    mov x0, #0 
    mov w8, #93 
    svc #0

msg:
.ascii "Hello, world! This message was created via ARMv8-A assembly code.\n"
len = . - msg
```

And then assemble (make sure to include the `-ggbd` flag to supply `gdb` with 
as much debugging information as possible) and link into an executable.

```sh
as -ggdb hello_world.s -o hello_world.o
ld hello_world.o -o hello_world
```
And then launch the debugger:

```sh
gdb -q -ex 'file hello_world' -ex 'layout split' -ex 'layout regs'
```

The flags supplied here to `gdb` are:

- `-q`: "quiet" -- do not print the introductory material.
- `-ex` COMMAND: execute a command within `gdb` on start up.
  - `-ex 'file hello_world'``: load the file `hello_world`.
  - `-ex 'layout split'`: display the source, assembly, and command windows in
    the TUI.
  - `-ex 'layout regs'`: when already in split layout display the register,
    assembler, and command windows in the TUI.

What we do in the debugger depends on what we are trying to observe.

For now, the focus will likely be on observing the effect of each assembly
instruction on the registers.

For this purpose, note the following commands in `gdb`:

- `stepi`: this performs the next one assembly instruction (and will step into
  functions in detail).
- `nexti`: this performs the next single assembly instruction (but won't step
  into function calls in detail, but perform the call and return to the next
  assembly instruction).

For the documentation related to this, see the section titled
["5.5 Continuing and Stepping"](https://sourceware.org/gdb/onlinedocs/gdb/Continuing-and-Stepping.html).

Don't step into operating system function calls if you are simply trying to
observe the effect of code that you have created.

Use the command `quit` to exit `gdb` when you are done.


