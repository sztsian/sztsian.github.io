---
title: 给想学 STM32 的朋友的一点资料 - 中文版
date: 2021-03-05T22:36:40+08:00
author: Zamir
layout: post
categories:
  - embedded
---

这篇面向中文读者。里面涉及到的资料部分是中文的。

## 为什么我要写这篇

两年前，我在做我的[家居自动化](https://sztsian.github.io/home-automation-1/)时，我开始觉得我应该学一点嵌入式编程。然而我太忙了（嗯，我知道这是个借口。。。）所以就一直没开工。

最近有位朋友问到我，是否可以提供一点学嵌入式开发的资料。我开始认真思考这个问题。虽然我现在仍然没有任何嵌入式开发经验，但是过去一段时间折腾家居自动化的过程中我了解了一些概念，所以大概不能算是个纯小白了。受 [The Best and Worst MCU SDKs](https://interrupt.memfault.com/blog/the-best-and-worst-mcu-sdks) 这篇文章的启发，我决定我先找出我心目中适合初学者的 MCU 应该有哪些特性。警告：这篇文章**非常主观**!

我希望

* 这个 MCU 的开发板和芯片非常容易买到
* 网上可以搜到充分的资料或教程
* 我能在任何主流操作系统上做开发
* 最好有个对新手非常友好的 IDE

由于我当然只考虑 ARM MCU, 因此这个范畴就不大了。当然，这些问题的答案可能在不同地区会有差异。

对于容易买到开发板/芯片这个因素，我首先想到的是
* 意法的 STM32 系列 MCU
* Nordic 的 nRF51, nRF52 系列
* 德州仪器的 cc26xx 系列
* 恩智普的 LPC 系列

而不幸的是，我能找到的 LPC 系列的资料实在太少。所以它就不符合我的第二个想法啦。而 cc26xx 系列 MCU 在 Linux 和 MacOS 下做开发会很麻烦，就又被排除了。对于 nRF51/nRF52 系列和 STM32 系列，两者都有很多资料，而且都可以在 Linux 以及 MacOS 下进行开发，所以看起来其实都不错。但不幸的是，当我首次尝试 nRF51 的时候，我花了相当长的时间才把点 LED 的例程跑起来，所以我对它的第一印象不太好（大概因为那时候我用的既不是 Nordic 官方开发板，也不是官方 IDE 吧。而且在 Linux 下做 nRF51 开发的资料不是特别容找，所以作为一个新手，我遇到了不少困难）。而用 STM32 点 LED 比我预想的简单太多，尤其是使用 STM32CubeIDE 进行开发的时候。所以我决定还是推荐 STM32 系列。

## 推荐的资料

我觉得，学嵌入式编程最好得能看一点简单的电路图。了解基本的外设和他们的特性可能对学习会有些帮助吧。再就是，对新手来说，从 IDE 使用入手，具有很清晰步骤的教程会更容易上手。此外，开发板当然也是不可或缺的。我写这些内容目的是给朋友提供一点信息，所以文中的内容也许不一定适合你（因为这些信息是我针对我们的需要筛选的）。

### 学看电路图

学嵌入式开发的最初，能看懂简单的电路原理图就够了。网上资料挺多的。非要我推荐的话，我觉得 [Beginner Electronics](https://www.youtube.com/playlist?list=PLah6faXAgguOeMUIxS22ZU4w5nDvCl5gs) 的第 1 到 21 集就挺有参考价值。注意它们是英文的。

### 学习使用 IDE

好的 IDE 能让开发更加容易。对于 STM32 开发的初学者，尤其是不用 Windows 的用户，我建议用 STM32CubeIDE 进行开发。STM32CubeIDE 是意法为 STM32 优化的开发环境。我觉得这篇 [博客](https://01001000.xyz/2020-05-11-Tutorial-STM32CubeIDE-Getting-started/) 介绍的挺适合新手阅读。如果你觉得这个还是太简单了，意法还在 Youtube 上发布了一系列 [MOOC - STM32CubeIDE basics](https://www.youtube.com/playlist?list=PLnMKNibPkDnFCosVVv98U5dCulE6T3Iy8)。这个系列除了介绍 STM32CubeIDE 之外，还演示了不同外设的基本配置。有人把这个视频[搬运到了哔哩哔哩](https://www.bilibili.com/video/BV1B7411d757?p=1)，方便大陆用户。

### 找到最适合你的资料

**警告：我并没有完整阅读下面推荐的资料。推荐基于我阅读的第一感觉**

我觉得 Donald Norris 写的图书《Programming with STM32: Getting Started with the Nucleo Board and C/C++》不错。我读了 Kindle 里的样张，看到这本书不仅从如何用 STM32CubeMX 生成代码开始讲解，还简单讲解了部分生成的代码是做什么功能的。
中文资料我首先推荐微雪的 [STM32CubeMX 系列教程](https://www.waveshare.net/study/article-629-1.html)。这其实算是朋友推荐给我的吧。然而这教程有个缺点：它是基于 STM32F7 系列制作的，如果你想买一样的开发板可能不是很便宜。
国内还有一些开发板厂商也做了不错的教程。比较成体系的有野火和正点原子两个厂商。他们有基于常见的 STM32F1 系列做的教程。避免做广告之嫌我就不放连接了。

意法自己的视频呢， [MOOC - STM32CubeIDE basics](https://www.youtube.com/playlist?list=PLnMKNibPkDnFCosVVv98U5dCulE6T3Iy8) 和 [MOOC - STM32CubeMX and STM32Cube HAL basics](https://www.youtube.com/playlist?list=PLnMKNibPkDnGtuIl5v0CvC81Am7SKpj02) 看起来还不错。

### 硬件

为什么我现在才讨论硬件？因为，作为初学者，一上来就买硬件可能会考虑不周（我刚开始就如此）。有时甚至会浪费钱。所以你选定了学习资料之后再找开发板至少会少走弯路（或者你也可以选择自带教程的开发板）。你身边如果有做嵌入式开发的朋友，那建议购买之前听下对方的建议。因为市面上的开发板实在太多了。如果你身边没有人了解嵌入式开发，那么我建议要么买意法自己的开发板，要么买有配套教程的开发板。

意法有较为便宜的 [Nucleo](https://www.st.com/en/ecosystems/stm32-nucleo.html) 系列开发板。它不算贵，使用也很方便，并且板子上自带官方调试器 - ST-Link 。因此你可以买到手直接插电脑上开始玩，而不用担心不了解如何使用调试器。然而它的板载外设少了点。意法的 [STM32 Discovery Kits](https://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-mpu-eval-tools/stm32-mcu-mpu-eval-tools/stm32-discovery-kits.html) 系列开发套件会多一点外设。意法称它“a cheap and complete solution for the evaluation of the outstanding capabilities of STM32 MCUs and MPUs”。
当然你也可以根据需要选择其他厂商的开发板。只要记得先搞清楚自己的需求。
