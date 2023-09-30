+++
title = "Set up for learning Armv8-A assembly via QEMU User Mode"
date = 2023-09-30
draft = true

[taxonomies]
categories = ["arm-asm"]
tags = ["emulation"]

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


```sh
sudo apt update -y && sudo apt upgrade -y
sudo apt install qemu-user qemu-user-static gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu binutils-aarch64-linux-gnu-dbg build-essential
```

```c
/* test.c */
#include <stdio.h>

int main(void) { 
    return printf("Hello! This program is executing ARMv8-A assembly.\n"); 
}
```

```sh
aarch64-linux-gnu-gcc -o test test.c
```

```sh
$ file test
test: ELF 64-bit LSB shared object, ARM aarch64, <snip>
```

```sh
qemu-aarch64 -L /usr/aarch64-linux-gnu ./test
```

```c
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



```sh
aarch64-linux-gnu-as hello_world.s -o hello_world.o
aarch64-linux-gnu-ld hello_world.o -o hello_world
./hello_world
```

## Debugging

```sh
sudo apt install gdb-multiarch qemu-user
```
In one terminal window, start the QEMU user mode process:

```sh
qemu-aarch64 -L /usr/aarch64-linux-gnu/ -g 1234 ./hello_world
```

And then connect to this process with the debugger in a separate terminal
window:

```sh
gdb-multiarch -q --nh -ex 'set architecture arm64' -ex 'file hello_world' -ex 'target remote localhost:1234' -ex 'layout split' -ex 'layout regs'
```

![Debugging Arm assembly via QEMU User Mode](/img/debug-arm-via-qemu-user.png)

