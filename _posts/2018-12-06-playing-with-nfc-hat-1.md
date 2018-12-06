---
title: Play with NFC HAT I
date: 2018-12-06T18:29:00+08:00
author: Zamir
layout: post
permalink: /playing-with-nfc-hat-1/
categories:
  - Uncategorized
---
The other day I got an NFC HAT for SBC to play with. And I started to play with it on my Raspberry Pi last week.

Things did not go smoothly, which is expected. But some part of it still goes beyond my expection.

So what's it? It's a NFC development board based on NXP PN7150. You can buy it [from taobao](https://item.taobao.com/item.htm?id=583055626971). It's header is compatible with Raspberry Pi, and minimal modification to use with [Salted Fish Pi](https://sbc-fish.github.io/sfpi/). As I already have Raspberry Pi 1/2/3, I simply plug it onto Raspberry Pi 2 running with Fedora.

As I have little knowledge with hardware, I started directly with reading the [documentation](http://www.nxp.com/documents/application_note/AN11697.pdf) and assume simply follow it will work, which is proven to be wrong.

So, I compiled [linux_libnfc-nci](https://github.com/NXPNFCLinux/linux_libnfc-nci), on my RPI2, and it simply do not work. Then I found the i2c-dev module is not loaded during boot time. Then I load the module and tried again, still do not work. And I just realized I don't have /sys/class/gpio on my system.

I checked the kernel config, no wonder, CONFIG_GPIO_SYSFS is not set. I searched and realized the GPIO sysfs is marked as deprecated in mainline kernel, so Fedora decide to disable it. Instead, the GPIO char device are suggested. However, the linux_libnfc-nci project do not support GPIO char device yet.

The first thing that came into my mind is to compile the support as a out-of-tree module. I did not realize it is not possible untile I tried 3 times, and then I start to check the dependency by manually `make oldconfig`. Hmm, it told me 'm' is not supported for this config.

As a result, I have to compile my own kernel temporary with the config enabled. And I have compiled a 4.19.7-300 one [here](https://zsun.fedorapeople.org/pub/pkgs/kernel.gpiosysfs/).

Now I can start play with it, and possible think of how to get char device supported.
