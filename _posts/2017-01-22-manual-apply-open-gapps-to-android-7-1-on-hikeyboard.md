---
id: 185
title: Manual apply Open GApps to Android 7.1 on HikeyBoard
date: 2017-01-22T22:12:36+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=185
permalink: /manual-apply-open-gapps-to-android-7-1-on-hikeyboard/
categories:
  - Notes
---
Today I decided to make my HikeyBoard an Android video box. So I need Google Play.

After downloading the Android binary, I just flashed it following

http://www.96boards.org/documentation/ConsumerEdition/HiKey/Installation/LinuxFactoryImage.md/

And then it comes to the OpenGApps. I downloaded the OpenGApps zip but unfortunately I can not reboot to recovery on HikeyBoard. So I decided to apply it manually. My final steps like the following.

enable developer options &#8211; USB debug.
  
adb kill-server
  
adb root
  
adb shell mount -o rw,remount /;
  
adb shell mkdir /tmp;
  
adb push * /tmp
  
adb shell mount -o rw,remount /system;
  
replace &#8216;install -d&#8217; with mkdir in META-INF/com/google/android/update-binary
  
offer variables a fixed value in META-INF/com/google/android/update-binary to make things easier
  
follow https://forum.xda-developers.com/galaxy-nexus/general/howto-apply-manually-update-to-4-0-4-t1571827