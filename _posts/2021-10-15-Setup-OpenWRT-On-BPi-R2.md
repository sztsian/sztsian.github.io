---
title: Setup OpenWRT on BPi-R2
date: 2021-10-16T23:05:10+08:00
author: Zamir
layout: post
categories:
  - embedded
---
It's pretty easy to get OpenWRT start and running on BPi-R2. However, I realized that I need to extend the root filesystem to the whole disk, which is where the struggling starts.

## Setup OpenWRT on BPi-R2

Just download the [bpi_bananapi-r2-squashfs-img.gz](https://downloads.openwrt.org/releases/21.02.0/targets/mediatek/mt7623/openwrt-21.02.0-mediatek-mt7623-bpi_bananapi-r2-squashfs-img.gz) from openwrt.org and dd it to the TF card like playing with any SBC. Replace `sdX` with the device name of your card.

```
$ wget https://downloads.openwrt.org/releases/21.02.0/targets/mediatek/mt7623/openwrt-21.02.0-mediatek-mt7623-bpi_bananapi-r2-squashfs-img.gz
$ sudo dd if=openwrt-21.02.0-mediatek-mt7623-bpi_bananapi-r2-squashfs-img.gz of=/dev/sdX status=progress
```

Then plug the card into BPi-R2. Make sure the power supply is firmly connected to BPi-R2 (it's a 12v power supply, by the way). Hold the PWR button until the red LED near the power cable lights up, and only the red LED on that area is on. It usually takes 10~15 seconds for me. Release the PWR button now. Soon you'll see the red LED near the power cable goes off and then only the red LED near the GPIO head is on. Now OpenWRT is up and running.

## Expand the root partition

This is where the confusion starts. Googling for a while I did not find anything useful. All of a sudden [this answer](https://forum.openwrt.org/t/expanding-openwrt-squashfs-image-sdcard/60606/5) came to my eye and I believe this is the way I should do. In short, the OpenWRT image for BPi-R2 contains 3 partitions. The last partition is not a single partition, but rather, a combination of the squashfs and the f2fs overlay. In order to extend the root partition, we need to extend the f2fs overlay, which requires us to extend the 3rd partition. Here is how I finish it.

* Install losetup in OpenWRT

```
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install losetup
```

* Check the current disk table 

```
/dev/root /rom squashfs ro,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,noatime 0 0
sysfs /sys sysfs rw,nosuid,nodev,noexec,noatime 0 0
cgroup2 /sys/fs/cgroup cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate 0 0
tmpfs /tmp tmpfs rw,nosuid,nodev,noatime 0 0
/dev/loop0 /overlay f2fs rw,lazytime,noatime,background_gc=on,discard,no_heap,user_xattr,inline_xattr,inline_data,inline_dentry,flush_merge,extent_cache,mode=adaptive,active_logs=6,alloc_mode=reuse,fsync_mode=posix 0 0
overlayfs:/overlay / overlay rw,noatime,lowerdir=/,upperdir=/overlay/upper,workdir=/overlay/work 0 0
tmpfs /dev tmpfs rw,nosuid,relatime,size=512k,mode=755 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,mode=600,ptmxmode=000 0 0
debugfs /sys/kernel/debug debugfs rw,noatime 0 0
none /sys/fs/bpf bpf rw,nosuid,nodev,noexec,noatime,mode=700 0 0
```

Then check the offset of the f2fs partition

```
root@OpenWrt:~# losetup
NAME       SIZELIMIT  OFFSET AUTOCLEAR RO BACK-FILE  DIO LOG-SEC
/dev/loop0         0 2883584         1  0 /mmcblk1p3   0     512
```

Now we see the offset is `2883584` in my case. Remember the number. Power off your BPi-R2 and take out your card. Now we need to connect the card to a running Linux and do the expand.

First let's expand the 3rd partition on the card. I use `cfdisk` to expand it. You can use any partition tools you are familiar with. 

After extending the partition, let's expand the f2fs 'partition' inside. I am using Fedora so I need to install `f2fs-tools` first. Here we need to use the offset `2883584` we get from the running OpenWRT just now.

```
$ sudo losetup --show -o 2883584 -f -P /dev/sde3 
/dev/loop0
```

Now the partition has been attached to the system as `/dev/loop0`. Let's try to extend it with `resize.f2fs /dev/loop0`. In my case, it fails with `[f2fs_do_mount:3526] Mount unclean image to replay log first` which I think is caused by removing the card when BPi-R2 is still running. Remounting it did not solve my issue. So I just created a brand new f2fs filesystem with

```
$ sudo mkfs.f2fs -f /dev/loop0

        F2FS-tools: mkfs.f2fs Ver: 1.14.0 (2020-08-24)

Info: Disable heap-based policy
Info: Debug level = 0
Info: Trim is enabled
Info: Segments per section = 1
Info: Sections per zone = 1
Info: sector size = 512
Info: total sectors = 15189504 (7416 MB)
Info: zone aligned segment0 blkaddr: 512
Info: format version with
  "Linux version 5.14.9-200.fc34.x86_64 (mockbuild@bkernel02.iad2.fedoraproject.org) (gcc (GCC) 11.2.1 20210728 (Red Hat 11.2.1-1), GNU ld version 2.35.2-5.fc34) #1 SMP Thu Sep 30 11:55:35 UTC 2021"
Info: [/dev/loop0] Discarding device
Info: This device doesn't support BLKSECDISCARD
Info: This device doesn't support BLKDISCARD
Info: Overprovision ratio = 2.330%
Info: Overprovision segments = 176 (GC reserved = 93)
Info: format successful
```

Detach the loopback device with `sudo losetup -D` and now it's time to plug the card into BPi-R2 and run again. You'll see your root filesystem is around the size you set by cfdisk.

