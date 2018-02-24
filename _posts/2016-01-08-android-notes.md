---
id: 161
title: Android Notes
date: 2016-01-08T17:50:37+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=161
permalink: /android-notes/
categories:
  - Notes
---
1. Change Resolution for Android on Dragon Board 410c

> fastboot oem select-display-panel adv7533_720p #physical resolution

If just want to act as a new resolution but not actually change it:

>     adb shell wm size 800x480

use `adb shell wm size reset to reset it.`

&nbsp;