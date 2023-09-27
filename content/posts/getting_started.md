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

There are some valuable resources out there for learning about bare metal
development on the Raspberry Pi. Here we focus on the Raspberry Pi 4, for which Adam
Greenwood-Byrne [GitHub: isometimes](https://github.com/isometimes) has written
an outstanding tutorial that documents his journey
[from bare metal to a Breakout game](https://github.com/isometimes/rpi4-osdev)
on the Raspberry Pi 4. We will take this as our starting point, but we will try
to uncover as much as we can of the "hidden" curriculum involved in this sort of
bare metal programming project as we go along.

<!-- more -->

## Raspberry Pi 4

The Raspberry Pi 4 presents some benefits and challenges by comparison with
earlier versions of the Raspberry Pi. Overall, it is a much more capable board
computer. Most importantly, it is available for purchase.

However, there is no readily available Raspberry Pi 4 machine available for 
emulation via QEMU: this means that there is more manual work to be done in
running and testing code in development. While a generic machine could be
used in QEMU, this would mean that the board-specific devices are not available.

## Background

Some knowledge of programming is assumed, in no particular language.

Some knowledge of computing architecture and the layers of abstraction involved
is also assumed, such as might be gained by the completion of the justly
renowned "From Nand to Tetris" course created by Noam Nisan and Shimon Schocken:

- The 'From Nand to Tetris' website: [https://www.nand2tetris.org/](https://www.nand2tetris.org/)
- Nisan, Noam, and Shimon Schocken. 2021. *The Elements of Computing Systems:
  Building a Modern Computer from First Principles*. Second edition. Cambridge,
  Massachusetts: The MIT Press.
  - [MIT Press](https://mitpress.mit.edu/9780262539807/the-elements-of-computing-systems/)
  - [Amazon](https://www.amazon.com/Elements-Computing-Systems-second-Principles/dp/0262539802)
- The Nand2Tetris courses on Coursera:
  - [Nand2Tetris Part I](https://www.coursera.org/learn/build-a-computer)
  - [Nand2Tetris Part II](https://www.coursera.org/learn/nand2tetris2)

But we will try to spell out assumptions and required knowledge as we go along,
and in particular we will try to define important terminology in the Glossary.

## Things you need

Required physical items:

- Development machine, preferably Linux (use WSL2 on Windows)
- Raspberry Pi 4 (BCM2711 SOC)
- USB to TTL serial cable
- SD card
- SD card reader (if not built-in to your computer)

Required software set-up:

- Development environment on your development machine, e.g. VSCode.
- Cross-compilation ARM tools for the AArch64 bare-metal target 
  (aarch64-none-elf):
    - [ARM Developer toolchain downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)

We will assume that these items are accessible and/or installed, but we will
walk through the actual set up to get things working step by step in the next
post.

## Resources

- The RPi4 OS dev Tutorial from Adam Greenwood-Byrne [GitHub: `isometimes`](https://github.com/isometimes)
  - Tutorial website: [https://www.rpi4os.com/](https://www.rpi4os.com/)
  - GitHub repo: [https://github.com/isometimes/rpi4-osdev](https://github.com/isometimes/rpi4-osdev)
- Related tutorials and how-tos, focused on Raspberry Pi 3 models:
  - [`rockytriton`'s Low Level Devel](https://github.com/rockytriton/LLD) and 
    [Low Level Devel YouTube playlist](https://www.youtube.com/playlist?list=PLVxiWMqQvhg9FCteL7I0aohj1_YiUx1x8)
  - [`s-matyukevich`'s Learning operating system development using Linux kernel and Raspberry Pi](https://github.com/s-matyukevich/raspberry-pi-os)
  - [`bztsrc`'s Bare Metal Programming on Raspberry Pi 3](https://github.com/bztsrc/raspi3-tutorial)

