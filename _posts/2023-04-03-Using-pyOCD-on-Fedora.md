---
title: Using pyOCD on Fedora
date: 2023-04-03T22:44:50+08:00
author: Zamir
layout: post

---

# Install pyOCD

```
pip3 install --user pyocd
```

# Config udev rules and MCU pack
## Add udev rules for your debugger

Download [the cmsis-dap udev rules](https://github.com/pyocd/pyOCD/blob/main/udev/50-cmsis-dap.rules) to `/etc/udev/rules.d/` . If it does not contain the USB VID/PID of your debugger, add a line similar to that. For exmaple, for WCH-Link with USB info like "1a86:8012 QinHeng Electronics WCH-Link"

```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="8012", MODE="666", GROUP="plugdev", TAG+="uaccess"
```

Then, run the following commands to make the rules take effect.

```
sudo udevadm control --reload-rules 
sudo udevadm trigger
```
## Download the Device Family Pack (DFP)
In my case, I am trying with MM32F3273. First I need to figure out the target name by

```
pyocd pack  find mm32f32
```
It will take a while to download all the pack info and provide the search result. Figure out the pack name and install it. In this case, my pack name is `MM32F3273G8P`.
```
pyocd pack install MM32F3273G8P
```

# Start using pyOCD

Now we can actually start using pyOCD.

To flash firmware into your board

```
pyocd flash -t MM32F3273G8P your-firmware.elf
```

To fire up gdbserver for debugging

```
pyocd gdbserver -t MM32F3273G8P
```

And then you can attach gdb to this server

```
[zsun@nas build]$ gdb your-firmware.elf 
...
(gdb) target extended-remote 127.0.0.1:3333
...
0x08000fce in GPIO_ClearBits ()
(gdb) bt
#0  0x08000fce in GPIO_ClearBits ()
#1  0x0800160a in main ()
```

To be continued.
