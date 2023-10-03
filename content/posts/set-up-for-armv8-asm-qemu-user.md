+++
title = "Setting up for learning Armv8-A assembly via QEMU User Mode"
date = 2023-09-30
draft = false

[taxonomies]
categories = ["arm-asm"]
tags = ["emulation", "linux"]

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

Learning about Armv8-A assembly when your main machine is x86 or the like
presents some extra hurdles.

One approach, of course, is simply to do the learning directly on a Raspberry Pi
with the standard `gcc` toolchains that are already in place. This gives direct
access to an Armv8-A (assuming that you are running a 64-bit operating system
on the Pi) processor, and you can experiment directly on the machine.

See this post for this approach using Raspberry Pi 4:

- [Setting up for learning Armv8-A assembly on RPi4](/posts/set-up-for-armv8-asm/)

Another approach is to use a fully fledged VM or full emulation via QEMU. These
approaches won't be covered here.

But QEMU can also be used in "user mode" as a light-weight way to learn how to
read and use Armv8-A assembly code on a main development machine that has a
different architecture.

## Assumptions and credits

The instructions below assume that the main machine is Linux, and that the
target is also Linux, but specifically `AArch64`.

Using Linux as the basis for this work is sensible, so as to take advantage of
the syscall infrastructure already in place, e.g. for writing to `stdout`.

These instructions owe a great deal to the following how-tos from Maria
Markstedter's Azeria Labs website:

- [https://azeria-labs.com/arm-on-x86-qemu-user/](https://azeria-labs.com/arm-on-x86-qemu-user/)
- [https://azeria-labs.com/debugging-with-gdb-introduction/](https://azeria-labs.com/debugging-with-gdb-introduction/)

## Set up QEMU ready for user mode compilation and debugging

First, set up the software infrastructure for cross compilation and using QEMU. 

```sh
sudo apt update -y && sudo apt upgrade -y
sudo apt install qemu-user qemu-user-static gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu binutils-aarch64-linux-gnu-dbg build-essential
```
Next install the tools needed for using the debugger to explore the effects of
your assembly code at the register level.

```sh
sudo apt install gdb-multiarch qemu-user
```
Create a test program in C to demonstrate that cross-compilation works.

```c
/* test.c */
#include <stdio.h>

int main(void) { 
    return printf("Hello! This program is executing ARMv8-A assembly.\n"); 
}
```
And compile the test program using the cross-compilation tool chain.

```sh
aarch64-linux-gnu-gcc -static -o test test.c
```
Running `file` on the executable that is output confirms that this is an Armv8-A
ELF for `AArch64`.

```sh
$ file test
test: ELF 64-bit LSB shared object, ARM aarch64, <snip>
```
Run the `AArch64` executable on your host machine using the QEMU user mode tool,
and specify the source of the libraries that would otherwise be missing from
the dynamic executable.

```sh
qemu-aarch64 -L /usr/aarch64-linux-gnu ./test
```

Alternatively, compile using the `-static` flag and `qemu-user-static` will,
behind the scenes, enable the binary to "just work" even though it is running on
the wrong architecture.

```sh
$ aarch64-linux-gnu-gcc -static -o test test.c
$ file test
test: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), 
statically linked, <snip>
```

Now create a test program in Armv8-A assembly.

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
Use the cross-compilation tool chain to assemble and then link the program into
an executable. Make sure to use the `-ggdb` flag to build debugging symbols into
the executable so that GDB can do its work more effectively.

```sh
mkdir build
aarch64-linux-gnu-as -ggdb hello_world.s -o build/hello_world.o
aarch64-linux-gnu-ld build/hello_world.o -o build/hello_world
./build/hello_world
```

## Start the debugger

Now we want to run the program, but in debugging mode, so that we can attach to
the process and step through the lines of assembly and observe the effect of the
assembly instructions on register state and on the output of the program.

In order to do this, we will open two separate terminal windows, one for the 
QEMU user mode process, and one for the GDB debugger.

In one terminal window, start the QEMU user mode process:

```sh
qemu-aarch64 -L /usr/aarch64-linux-gnu/ -g 1234 ./build/hello_world
```

And then connect to this process with the debugger in a separate terminal
window:

```sh
gdb-multiarch -q --nh -ex 'set architecture aarch64' \
-ex 'file build/hello_world' -ex 'target remote localhost:1234' \
-ex 'layout split' -ex 'layout regs'
```

You should now see the GDB TUI interface, including the registers.

![Debugging Arm assembly via QEMU User Mode](/img/debug-arm-via-qemu-user.png)

What we do in the debugger depends on what we are trying to observe.

For now, the focus will likely be on observing the effect of each assembly
instruction on the registers.

For this purpose, note the following commands in `gdb`:

- `stepi`: this performs the next one assembly instruction (will step into
  functions).
- `nexti`: this performs the next single assembly instruction (but won't step
  into function calls).

Don't step into operating system function calls if you are simply trying to
observe the effect of code that you have created.

Use the command `quit` to exit `gdb` when you are done.
