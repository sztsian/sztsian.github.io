---
title: Why to Start With ESP32
date: 2022-01-06T21:17i53:19+08:00
author: Zamir
layout: post
categories:
  - embedded
---
**Disclaimer: I'm just a beginner of learning embedded, I wrote all of my embedded articles based on the view of myself as a beginner, so different ideas from embedded professions are highly unavoidable.**

Last spring, I've wrote about start [learning embedded with STM32](https://sztsian.github.io/embedded/2021/03/04/Notes-for-beginning-with-STM32-English.html). However, soon after I wrote that, I realize that the price of STM32 series has gone up to an incredible level. Which makes me wonder if that's still a good choice.

Recently I started to think about the beginner's question again. This time, I have some different thoughts: if the learning material can be more related with daily life, the learner will take it much easier. Nowadays IoT is definitely a hot topic. So if the MCU can have some sort of IoT capabilities, it will definitely make the learner happier. So MCU with real wireless communication function is a better choice. Currently, there are a bunch of wireless protocol in real life. The most widely mentioned including (but not limited to)

* [WiFi](https://en.wikipedia.org/wiki/Wi-Fi)
* [Bluetooth](https://en.wikipedia.org/wiki/Bluetooth) and [Bluetooth Low Energy (aka BLE)](https://en.wikipedia.org/wiki/Bluetooth_Low_Energy)
* [Zigbee](https://en.wikipedia.org/wiki/Zigbee)
* [LoRa and LoRaWan](https://en.wikipedia.org/wiki/LoRa)
* [NB-IoT](https://en.wikipedia.org/wiki/Narrowband_IoT)
* or even the traditional 433MHz communications with OOK and ASK

and more.

In my option, it would be much easier for the beginner if he or she can just connect the MCU to other existing devices, especially mobile phone and laptop. So WiFi and Bluetooth (including BLE) outstands others of all those listed.

So this time, the criterias are

* The chips or boards should be easily available.
* Tutorials should be easily available.
* People should be able to develop and debug for the MCU on any major OS (Linux, MacOS, Windows).
* There is an IDE that is easy to use even for people who do not have embedded experience.
* **The chip should have WiFi or BLE**
* **Only 32bits MCUs**

There aren't much choice matching such criteria, especially in where I live. I already mentioned why I do not like cc26xx and nRF5x for beginner, then there are only ESP8266 series, ESP32 series, or WinnerMicro [w600](http://www.winnermicro.com/en/html/1/156/158/497.html). Of all these, ESP8266 and ESP32 series have the best user community, and materials are most widely available online. But ESP8266 seems to be not recommended for new design (NRND), I think ESP32 series outstands othere.

So let's talk more about ESP32.

ESP32 is a series of wireless MCU produced by Espressif. As the time of writing, they are

* ESP32, which contains Xtensa 32-bit LX6 microprocessor(s) with 2.4G WiFi and Bluetooth 4.2 BR/EDR and BLE support.
* ESP32 S2, which contains a Xtensa 32-bit LX7 microprocessor with 2.4G WiFi
* ESP32 S3, which contains Xtensa 32-bit LX7 microprocessors with 2.4G WiFi and Bluetooth 5(LE)
* ESP32 C3, which contains a 32-bit RISC-V microprocessor with 2.4G WiFi and Bluetooth 5(LE).

What's more, Espressif provides [Arduino support](https://github.com/espressif/arduino-esp32) for [ESP32, ESP32 S2, ESP32 C3](https://docs.espressif.com/projects/arduino-esp32/en/latest/getting_started.html#supported-soc-s). In case you are a newbie reader, Arduino is an open-source hardware and software company, its Arduino IDE is famous for it being easy to use. I hear that even artists can use Arduino to do some embedded related artists without much trouble.

Now, people can freely decide to use ESP32 with Arduino which is much easier to start with, or to use ESP-IDF framework provided by Espressif directly. Espressif even wrote good get started guide for both [Arduino-ESP32](https://docs.espressif.com/projects/arduino-esp32/en/latest/getting_started.html) and [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html).

There are so many ESP32 boards available on the internet. The biggest difference for them are mostly only about periphrals. So just purchase as you prefer. Or if you don't know what to start with, buying a minimal ESP32 board like Node32S also should work, as it should be pretty chip (less than USD $3 here).
