---
id: 28
title: Working under https-proxy
date: 2014-08-11T11:06:19+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=28
permalink: /working-under-https-proxy/
categories:
  - fedora
---
1. SSH over http proxy
  
for Fedora:
  
#yum install connect-proxy
  
then configure ~/.ssh/config

> Host 202.190.\*.\*
  
> ProxyCommand connect-proxy -H proxy.corp.com:80 %h %p
  
> ServerAliveInterval 30

then you can ssh user@202.194.1.2 now

For Solaris 11, you just need to configure your ssh config.

> HOST 202.190.\*.\*
  
> ProxyCommand /usr/lib/ssh/ssh-http-proxy-connect -h proxy.corp.com -p 80 %h %p

See http://docs.oracle.com/cd/E19683-01/817-0365/6mg5vpmga/index.html for more information and if you are SOCKS proxy.

2. IMAP/SMTP over http proxy

Download http://proxytunnel.sourceforge.net/ and then compile it

copy the binary file to somewhere such as /usr/local/bin
  
then run the following
  
#proxytunnel -a 993 -p proxy.corp.com:80 -d imap.gmail.com:993 &
  
#proxytunnel -a 465 -p proxy.corp.com:80 -d smtp.gmail.com:465 &

Configure Thunderbird config to localhost:{smtp,imap}port

3. Maven over http proxy

Find the maven config. If you use yum install maven to install under Fedora, it will be in /etc/maven/settings.xml.
  
Fix the section. Add like the following

>  <proxy>
  
> <id>optional</id>
  
> <active>true</active>
  
> <protocol>http</protocol>
  
> <host>proxy.corp.com</host>
  
> <port>80</port>
  
> </proxy>

4. OT. Get Emoji characters display in Fedora:

> <pre>sudo yum install gdouros-symbola-fonts</pre>

See http://fedoramagazine.org/how-to-get-emoji-to-display-on-fedora/