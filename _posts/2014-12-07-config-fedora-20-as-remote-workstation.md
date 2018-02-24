---
id: 67
title: Make Fedora 20 a remote workstation
date: 2014-12-07T16:15:08+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=67
permalink: /config-fedora-20-as-remote-workstation/
categories:
  - fedora
---
Some guys in campus want to find a Linux workstation to use, but they just do not want to modify his own disk. So I decide to configure one of my workstation as a remote workstation for them to share.

I searched a lot for how to configure VNC with xdmcp but it did not work fine for me. I suddenly thought that since they are using Windows, so they must have rdp. So I switched my idea to use xrdp as the resolution.

The steps is easy, but due to my poor network connection, it takes me some time to test.

1.Install XRDP

yum install xrdp

2. Make XRDP enable by default

systemctl enable xrdp.service
  
systemctl start xrdp.service
  
systemctl enable xrdp-sesman.service
  
systemctl start xrdp-sesman.service

3. Add firewall rules to allow rdp access.
  
firewall-cmd &#8211;permanent &#8211;add-port=3389/tcp

4.If you want to configure a different Desktop environment, check if you have a the &#8220;PREFERRED=startxfce4&#8243; line in /etc/sysconfig/desktop. You can replace startxfce4 with other Desktop environments by your wish.