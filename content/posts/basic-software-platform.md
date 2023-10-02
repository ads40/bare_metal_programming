+++
title = "A basic software platform for the Raspberry Pi 4"
date = 2023-09-28
draft = false

[taxonomies]
categories = ["rpi4-bare-metal"]
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

In order to set up the most basic software platform that can run at the lowest
level on the Raspberry Pi 4, we need first to consider a number of things:

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

We will cover each of these topics here at a general level in order to
familiarise ourselves with the terrain.

Subsequent posts will then break down and set out the detailed steps required to
implement this foundational platform.

## What happens when the Raspberry Pi 4 is turned on?

There is some documentation for this provided by Raspberry Pi:

- [Raspberry Pi 4 Boot Flow](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-4-boot-flow)

First, there are a couple of locations for firmware code and configuration
options that we need to be aware of.

Second, we need to be aware that there is a layer of closed source firmware both
on the device's board itself and then in the boot folder of the SD card. We
can't modify this firmware and it is necessary to start the device.

Instead, we will begin our custom layer of software from the point where the
device's firmware first loads and then runs the kernel image.

### ROM and Boot EEPROM

The lowest level of firmware is stored on the machine in ROM, and it is run
directly from the ROM by the GPU on the device when the power is turned on.

Whereas earlier Raspberry Pi models then used a binary file `bootcode.bin`
located on the boot partition of the SD card for the next stage of the boot
process, the Raspberry Pi 4 has an
[EEPROM](https://en.wikipedia.org/wiki/EEPROM) on board that contains the
firmware used in the second stage of booting the system and initial
configuration options.

We don't need to touch the EEPROM firmware or configuration. Assuming that we
have previously booted the Raspberry Pi 4 from the Raspberry Pi OS SD card, then
the EEPROM will have been automatically updated as required by the Raspberry Pi
OS: "Raspberry Pi OS automatically updates the bootloader for critical bug
fixes."
([Raspberry Pi 4 Boot EEPROM](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-4-boot-eeprom)).

### `config.txt`

Most computing devices feature some sort of BIOS as a component of the initial
boot process, but Raspberry Pi devices use a optional text file named
`config.txt` located on the boot partition of the SD card.

This file is read by the firmware running on the Raspberry Pi's GPU during the
boot process.

We will later need to set a configuration option in `config.txt` in order to
achieve reliable serial communication via UART.

We won't need to set any other options for now.

### The boot sequence

The initial boot sequence for the Raspberry Pi 4 looks like this, in general
terms.

1. Device powers on.
2. In the first stage, the GPU starts and runs the initial layer of firmware
   stored in ROM on the device itself.
3. Assuming normal operation (not in recovery mode), the firmware running on the
   GPU then reads the EEPROM for the second stage firmware and runs it.
4. In the second stage, the firmware running on the GPU:
    - Initialises the device's clocks and SDRAM.
    - Reads the configuration stored in the EEPROM.
    - Proceed with the next phase of the boot process based on the `BOOT_ORDER`
      parameter stored in the EEPROM.
5. In normal operation, the boot mode will be `SD CARD`, and so the firmware
   will then attempt to load further firmware and configuration from the SD
   card. These include:
    - The firmware `start4.elf`, in normal operation (there are other
      `start4*.elf` files that provide more limited configurations of components
      on the Raspberry Pi 4 device).
    - The `config.txt` file.
    - The `cmdline.txt` file.
    - 'Device Tree blob' files (`.dtb`) that contain hardware definitions
      for the components available on the device: these will be used to set up
      the Raspberry Pi OS Linux kernel in normal operation.
6. The `start4.elf` firmware binary is loaded from the SD card and runs on the
   GPU; this layer of firmware takes over the boot process at this point, and
   among other things seeks to start the CPU cores and load a kernel image for
   the operating system.
7. This is the point at which our custom kernel image can come into the picture.

More information about the contents of the boot folder is given the Raspberry Pi
documentation:
[Boot Folder Contents](https://www.raspberrypi.com/documentation/computers/configuration.html#the-boot-folder).

## What about `kernel8.img` and Arm-specific assembly language?

The kernel is the fundamental software layer running on the device, after the
device's firmware has completed its initial bootstrapping set-up. For a given
operating system, the
[kernel intermediates the hardware layer from application code](https://en.wikipedia.org/wiki/Kernel_%28operating_system%29).
The kernel that we develop for the Raspberry Pi 4 will be very limited.

The default name expected for the kernel image that the firmware loads on the
Raspberry Pi 4 is `kernel8.img`, targeting the Armv8-A 64-bit Instruction Set
Architecture (ISA), because the `arm_64bit` configuration option defaults to `1`
on the RPi4 and the device therefore defaults to booting into 64-bit mode.

As such, `kernel8.img` is the file that we will be producing a custom version of
as the base layer for building our custom operating system and software.

But: we still have nothing in place that is remotely like an operating system.
The device is "on" but the only way to communicate with it is via native
assembly instructions and with some crucial parameters at hand.

For the RPi4, native assembly means the Armv8-A 64-bit ISA, also known as 
`AArch64`.

We will need to learn a little about Armv8-A assembly and some device-specific
layout parameters in order to get a custom kernel image working.

For an accessible introduction to Armv8-A assembly, work through the Armv8
chapter of [Dive into Systems](/glossary/dive-into-systems/):

- [Dive Into Systems > 9. ARMv8 Assembly](https://diveintosystems.org/book/C9-ARM64/index.html)

We work through this material in the Arm v8-assembly project.

However, much of the programming at this level is then done in C, as we build on
top of this initial kernel layer written in assembly, but in a much easier to
read and write format.

## How does a cross compilation tool chain work?

TODO

## Next steps

TODO
