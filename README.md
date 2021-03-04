# CBM-II 8088 CPU + RAM expansion board

## About the project

This repository is the hardware part of the project to create a PC compatibility system for Commodore CBM-II computers. The software counterpart is located at:

https://github.com/MichalPleban/cbm2-pc-emulator

More information about the project can be found at:

http://www.cbm-ii.com/

## About the board

The board is a replica of the original 8088 processor board for Commodore CBM-II computers. The replica is 100% compatible with the original card, so you can still run original MS-DOS 1.25 or CP/M-86 1.1 on it.

Several additional features were added to the card which allow for building a system that has enough power to run modern versions of DOS and DOS software:

### Fixed stability problems

The original card has serious stability problems in many machines, and it simply refuses to run on most low-profile CBM-II computers. These problems are caused by noise on /ERAS and /ECAS signals which cause memory corruption. Depending on the computer motherboard and/or PLA, the card may either work without any problems, hang once every few minutes or simply not work at all.

These problems are removed in the replica, and it works in all CBM-II computers.

### 1 MB memory expansion

The expanded memory is visible to both CPUs and allows DOS to access full 640 kB of memory – running DOS in only 256 kB leaves too little room for almost any application. The 64 kB segments above the 640 kB barrier can be individually turned off to allow other hardware (such as ISA graphics cards) to map into the address space.

The card has also the cartridge ROM with “fast boot” KERNAL modifications, thanks to which the memory test during powerup is greatly shortened (the test of 1 MB memory with the original would take around 2 minutes – this is shortened to mere 4 seconds).

### 12 MHz system clock

The card is equipped with a clock generator that produces either 12 MHz system clock with 50% duty cycle (for the NEC V20 processor) or 8 MHz clock with 33% duty cycle (for the 8088 processor). Compared with the 5 MHz on the original card, it gives a much faster system.
The system speed is further increased by using SRAM chips, which do not require refresh. Therefore the memory refresh system does not steal cycles from the CPU like on the original card.

### Hardware SPI interface

The SPI system is used to create a fast interface to a SD card, which serves as a 8 GB hard disk. Because the interface is implemented in hardware using shift registers, in combination with the INS/OUTS instructions of the NEC V20 CPU, it allows for theoretical read and write speed exceeding 1 MB / s.

### Battery backed RTC clock

The clock keeps the date and time for DOS and has also a few bytes of battery backed NVRAM to hold persistent configuration data.

### Reprogrammable EEPROM

The PC compatibility software is stored in an EEPROM chip, which can be reprogrammed from the 8088 side. Thanks to this, The library can be upgraded just by putting a newer version on the SD card and running the upgrade program from within DOS.

Having the whole software in the EEPROM allows also for diskless boot – the whole software is loaded from ROM, not from the disk like with the original card.

### Expansion port

The expansion port contains all the CPU bus signals, which should make it possible to add an ISA interface in the future. A smaller port with just the SPI and I2C signals is also available for additional hardware expansions in the future.

### Support for hardware virtualization

The card can generate NMI signal whenever an I/O port is accessed by the 8088 CPU. The I/O port access and the value being written to it are stored in hardware latches, so that the NMI routine can determine which I/O port was accessed and perform appropriate actions to simulate the hardware.

A video memory access latch was also added, which holds information about which parts of the video buffer at segment 0B000h was modified. This information speeds up screen refresh, because only the modified parts are being refreshed.

### 8087 coprocessor compatibility
The original card had a bug, due to which using the 8087 FPU was not possible – the card would lock up when the FPU was inserted into its socket. This bug has been fixed in the replica.

### Unobtainable chips replaced

The original card uses the 6525 Tri-Port Interface chip which is impossible to obtain. It has been replaced by an 8255 chip.

### Cartridge ROM sockets
The card has sockets for cartridge ROM or RAM at $4000, $6000 and $C000 which allow using  existing cartridge images. If a cartridge ROM is present, the card boot menu allows selecting whether to boot into BASIC or start the cartridge.

## Building the card

The PCB of the board has 4 layers to make it smaller and fit into the small space inside the low-profile CBM-II machines. Check with your PCB factory if they can make a 4 layer PCB for you.


The board contains an Atmel ATF1508 CPLD chip. You need a suitable programmer, such as ATF15xx-DK2, to program the chip. The board has a JEDEC connector which can be used to connect to the programmer directly. Use the cobalt.jed file from the repository to program the chip.

There are two ROM images in the archive which are necessary for the card to work:
-	8088.bin – 27C512 chip image to be placed at U7D
-	6509.bin  - 27C64 chip image to be placed at U2C

There are also chargen_lp.bin and chargen_hp.bin images which can be used to replace the character ROM in your machine (correspondently for low-profile and high-profile machines). They contain an alternate character set with some characters that are useful when running DOS (such as the backspace character). The card must be configured to use this character set – press F10 after booting the card.

The freedos.zip archive contains an image of a bootable FreeDOS hard disk. Put this image onto an SD card that is at least 8 GB of size (there are several SD card imaging applications available, I am using Win32 Disk Imager). After writing the images, the SD card will be readable on your PC, so you can easily add other applications. Note that you must use this image to boot the 8088 card, because the SD card must be formatted with special geometry for the INT 13h functions in the 8088 BIOS.

The ROM image from the repository contains the current version of the PC compatibility library. If you want to update the library in the future, grab the upgrade.com file from the repository, put it on the SD card and run it in DOS – it will write the newest version to the EEPROM chip on the board.
