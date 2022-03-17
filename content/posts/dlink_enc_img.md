---
title: "Dlink encrpted_img extractor"
description: "EMBA is now able to decrypt dlink-firmware with the encrpted_img header"
date: 2022-03-17T06:46:07-04:00
draft: true
tags:
  - extractor
  - encrypted
  - dlink
  - EMBA
---

`Dlink` network devices, especially routers are an fairly common sight, which have a history of static encryption algorithms.
So we wanted to expand EMBAs decryption capabilities to include devices like the DIR-X1560, DIR-X5460, DIR-X4860, etc.


The Goal:
---
Find and reproduce the decryption algorithm used in DLink-Firmware with the Header `encrypted_img` followed by the firmware size.

Idea:
---
Buy a cheap DLink-router that uses that specific firmware encryption and revers engineer the encryption by obtaining a decrypted firmware-image/root-filesystem.

The Process:
---
1.	Disassembly (HW):
    In the search of a “hidden” (not externally accessible) interface or to directly access the nand-flash.

**PICS TODO**

And that looks like a UART Interface.

2.	UART identification:
    There are different ways to identify the Pin-layout of unlabeled UART-Interface, tools like JTAGulator or manually. But when GND and the Voltage-level is identified correctly, baudrate and rx/tx pins, can - if in question - just be found by trial and error.

3.	Where to get the root-fs:
    Now that we have access to the device and its debugging options, we try to look for the easiest/fastest way to dump the flash contents or the root-fs directly.

    First Target was the login-shell of the device, which unfortunately requires credentials.
    Next in line was the bootloader itself:

    
4.	Flash-dump:
    One easy way to readout Flash-memory is via  https://github.com/Depau/bcm-cfedump, which uses the cfe-bootloders dump command and saves the ASCII-hex data back into a binary file.
    
5.	Flash-content analysis:
    There are 2 root-fs in the flash.
    The factory-default or “backup” and the current root-fs.
    To analyze the flash-dump we just throw it into EMBA and scan the results.

6.	Interesting binaries:
    fota, fota_config, fota_output, prog.cgi are interesting elf-files and had to be manually analyzed with Ghidra.

7.	The Treasure/Solution:
    When reversing library functions used in those elf-files, it becomes apparent, that DLink uses a static decryption-algorithm to decode its firmware-updates.
    (gj_decode() in the cms_util.so library)

8.	Bash implementation:
    After understanding the whole process from encrypted binary to decrypted flash-image, which decrypts the binary in 128kiB blocks and removes header and some trailing bytes.
    Following along step by step produces a UBI file and EMBA then extracts and analyzes its contents.
    
