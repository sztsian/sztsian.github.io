---
title: Home Automation I
date: 2019-01-14T22:29:00+08:00
author: Zamir
layout: post
permalink: /home-automation-1/
categories:
  - Uncategorized
---

I've been thinking about to automating my living for quite some time.  Basically, my requirements are:
* I can control the power of some home appliances no matter which platform I am using, be it Linux or Android or even iOS. And it can be controlled with customized rules.
* I can power on my workstation without using WOL.

For the first requirement I've purchased some so-called smart power strip in the early days. But unfortunately it has a lot of limitations and they are not really 'smart'. Most of them only work with a timer. So I've been looking for alternatives.

Well for the second requirement, I've been thinking about adding a relay to control the power button. However after some discussion with Shankerwangmiao and z4yx, they told me they already have made some product level prototype, and Shankerwangmiao kindly offered me the PCIe adapter they made for free. They call it IPMI_TU.

During the new year holiday, I decide to spend more time into the first requirement. And Sonoff comes up to me. Their products use an app called EWeLink which seems to have more features than the ones I have. After some research, I know Sonoff products are equipped with a SoC called ESP8266, which is very popular recently in the so-called 'IoT' area. I even found an open source firmware for a series of Sonoff products called [Sonoff Tasmota](https://github.com/arendst/Sonoff-Tasmota) which is appealing to me. The Sonoff Tasmota firmware supports controlling using MQTT, which is a plus as I can make customized rules anywhere and just make the MQTT call when the rules met. The Sonoff Tasmota firmware also works on many other ESP8266 based smart plug, so I checked their list and finally come up with one of the smaller sized variant and a Sonoff Basic smart switch to control my light.

Now it's the time to flash them. I think it is easy, but in fact, it really takes me some time.

In order to learn about the Arduino IDE and ESP8266 flashing, I purchased a NodeMCU board in advance. The Arduino IDE is [available in Fedora](https://src.fedoraproject.org/rpms/arduino) so I only need to do `dnf install -y arduino`. But this is just the beginning. To make a long story short, I need to install a bunch of Arduino libraries which is not a problem first, but result in I need to choose some older version library to workaround bugs in newer ones.

So here are some notes when I try to flash the [Huafan smart plug](https://github.com/arendst/Sonoff-Tasmota/wiki/HuaFan-Smart-Socket) I purchased.

* Crystal Frequency needs to be changed to 40MHz
* Choose `v1.4 Prebuild` for IwIP Variant to workaround some bug
* Always connect GPIO0 to ground before power the ESP8266 on for flashing. It can be disconnected after powering up, but it won't work if you power the ESP8266 on then connect GPIO0.

The first is pretty straight forward, but the other 2 notes really took me a long debugging time before I know the expected way.

Thanks for imi415's suggestions on narrowing the WiFi problem down.

And for flashing the [Sonoff Basic switch](https://github.com/arendst/Sonoff-Tasmota/wiki/Sonoff-Basic):

* Remember to change Crystal Frequency back to 26MHz
* The IwIP Variant still need to be `v1.4 Prebuild`
* Disable some unessential features if you don't need by editing `my_user_config.h`

That's pretty much of it for the two.

Then it comes to the IPMI_TU. IPMI_TU is not ESP8266 based. Instead, they use STM32F103 which is an ARM Cortex-M3 MCU, and a WIZnet W5500 ethernet controller. To flash an STM32 a tool called [stlink](https://src.fedoraproject.org/rpms/stlink) is needed, which is also available in Fedora as `stlink` or `stlink-gui`.

Since IPMI_TU originally designed for other use cases, they use [Protocol Buffers](https://developers.google.com/protocol-buffers) as the data serialization protocol in their [firmware](https://github.com/z4yx/NanoIPMI-FW). This is overkill for my use case, so I replaced the control function with a plain text one.

When it comes to flashing, I did something wrong in the beginning, and after that, I cannot flash it again using the open source variant of stlink. I figured out a way to re-flash its bootloader and used the official STM32 flashing tool to flash. Luckily after that, the STM32 is back to normal.

One more thing to note, the firmware of IPMI_TU uses the [DHCP log server option](https://tools.ietf.org/html/rfc2132) as a hackerish way determine the MQTT, so changes to the DHCP configuration is needed.

Now the firmware part is done. I'll write my experience about the server side later on.
