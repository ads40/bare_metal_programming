+++
title = "Getting started with Raspberry Pi 4 bare metal development"
date = 2023-09-26
draft = true

[taxonomies]
categories = ["rpi4"]
tags = ["bare-metal"]

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

There are some valuable resources out there for learning about
[bare metal](/glossary/bare-metal/) development on the Raspberry Pi. Here we
focus on the Raspberry Pi 4, for which Adam Greenwood-Byrne
([GitHub: isometimes](https://github.com/isometimes)) has written an outstanding
tutorial that documents his journey
[from bare metal to a Breakout game](https://github.com/isometimes/rpi4-osdev)
on the Raspberry Pi 4.

We will take this as our starting point, but we will try to uncover as much as
we can of the "hidden" curriculum involved in this sort of bare metal
programming project as we go along.

<!-- more -->

## Raspberry Pi 4

The Raspberry Pi 4 presents some benefits and challenges by comparison with
earlier versions of the Raspberry Pi.

In terms of benefits, it is a much more capable board computer. And, most
importantly, it is more readily available for purchase.

However, there is no readily available Raspberry Pi 4 machine available for 
emulation via QEMU: this means that there is more manual work to be done in
running and testing code in development. While a generic machine could be
used in QEMU, this would mean that the board-specific devices are not available.

We will therefore focus on running all our software directly on the Raspberry
Pi 4 itself.

## Background

Some knowledge of programming is assumed, in no particular language.

Some knowledge of computing architecture and the layers of abstraction involved
is also assumed, but only at a relatively low level, such as might be gained by
the completion of [the justly renowned "From Nand to Tetris" course and textbook
created by Noam Nisan and Shimon Schocken](/glossary/nand-to-tetris/).

But we will try to spell out assumptions and required knowledge as we go along,
and in particular we will try to define important terminology in the Glossary.

## Things you need

Required physical items:

- Development machine: Linux or WSL2 on Windows is assumed for this tutorial.
- Raspberry Pi 4 (BCM2711 SOC) which runs an [ARM Cortex-A72 processor](/glossary/arm-cortex-a72/).
- USB to TTL serial cable
- SD card
- SD card reader (if not built-in to your computer)

Required software set-up:

- Development environment on your development machine, e.g. VSCode.
- Cross-compilation tools:
  - On Linux or WSL2 on Windows, use the ARM developer toolchain for the AArch64
    bare-metal target (aarch64-none-elf):
    [ARM Developer toolchain downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
  - On MacOS, XCode will need to be installed (and with it `clang`) and then
    LLVM should be installed -- however, this tutorial does not address the use
    of these tools.
- Install a bootable image of Raspberry Pi OS on the SD card:
  - Use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to 
  - Then make sure that the Raspberry Pi 4 boots with the SD 

From this point, it will be assumed that these items are accessible and/or 
installed, in order to focus on the actual set up to get things working step by
step in the next post.

## The next steps

The critical next steps are:

1. To set up the most basic software platform that can run at the lowest level
   on the Raspberry Pi 4.
2. To enable two-way communication with our development machine, operating at 
   a very low level, so that we can start to interact with the software that we
   are developing for the Raspberry Pi 4.

In order to do the first item above, we need to know something about the
following items:

- How the Raspberry Pi 4 machine is designed to behave when it is first turned
  on.
- How to program for the Raspberry Pi 4 machine when it is operating at the
  earliest stage of its kernel boot process, using ARM-specific assembly
  language.
- How to run the cross compilation tool chain on our development machine and
  then transfer the resulting compiled programs to the SD card and then into
  the Raspberry Pi.
- How to set up the initial version of our kernel -- this is program that will
  effectively control our computer's operation once the initial bootstrapping
  program has completed.

In order to complete the second item above, we need to know something about
these further topics:

- How UART can be configured on the Raspberry Pi 4 via software and via physical
  connection to the correct GPIO pins on the Raspberry Pi 4 device.
- How to configure and run software on our development machine in order to be
  able to communicate via UART with the Raspberry Pi 4.

There is a lot of material here, but it is worth taking the time to set out and
explain each element properly, to ensure a solid foundation for the rest of the
work that we will be pursuing.

## Resources

As always, we stand on the shoulders of giants. These materials are heavily
dependent on the following published tutorials and how-tos.

- The RPi4 OS dev Tutorial from Adam Greenwood-Byrne [GitHub: `isometimes`](https://github.com/isometimes)
  - Tutorial website: [https://www.rpi4os.com/](https://www.rpi4os.com/)
  - GitHub repo: [https://github.com/isometimes/rpi4-osdev](https://github.com/isometimes/rpi4-osdev)
- Related tutorials and how-tos, focused on Raspberry Pi 3 models:
  - [`rockytriton`'s Low Level Devel](https://github.com/rockytriton/LLD) and 
    [Low Level Devel YouTube playlist](https://www.youtube.com/playlist?list=PLVxiWMqQvhg9FCteL7I0aohj1_YiUx1x8)
  - [`s-matyukevich`'s Learning operating system development using Linux kernel and Raspberry Pi](https://github.com/s-matyukevich/raspberry-pi-os)
  - [`bztsrc`'s Bare Metal Programming on Raspberry Pi 3](https://github.com/bztsrc/raspi3-tutorial)

