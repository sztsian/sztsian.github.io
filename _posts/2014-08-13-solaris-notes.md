---
id: 34
title: Solaris notes
date: 2014-08-13T16:06:09+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=34
permalink: /solaris-notes/
categories:
  - Notes
---
1. Enable rsh
  
svcadm enable svc:/network/login:rlogin or inetadm -e rlogin
  
svcadm enable shell:default

2. Configure rsh/rlogin with no password
  
vim ~/.rhosts , add like this
  
`hostname`
  
or
  
`hostname username`

3. Enable root login via rlogin

`vi /etc/default/login`
  
then comment the following line
  
`CONSOLE=/dev/console`
  
this line is to restrict root to use /dev/console only. So if this is set, root can not login from telnet/rlogin

4. Enable root login via SSH
  
Firstly, we need to fix the ssh config (/etc/ssh/sshd_conf) to PermitRootLogin yes.
  
In Solaris 11, root is defined as a role in /etc/user_attr, so if we need to ssh as root, we need to modify this file and then change root::::type=role to root::::type=normal (if this line exists). See the following links.

http://www.oracle.com/technetwork/articles/servers-storage-admin/o11-112-s11-first-steps-524819.html

http://docs.oracle.com/cd/E23824_01/html/821-1473/user-attr-4.html

5.The package &#8216;entire&#8217;
  
The summary of the package &#8216;entire&#8217;: incorporation to lock all system packages to the same build
  
It do not consist of any file, just the meta data of all package of the current build

6. Show total memory
  
prtconf | grep &#8220;Memory&#8221;

7. Show CPU information
  
psrinfo -pv(physical and virtual)
  
psrinfo -p (the total number of physical core)

8. Quit a zone
  
~.
  
see http://docs.oracle.com/cd/E19044-01/sol.containers/817-1592/fpcft/index.html

9. Quit a ILOM
  
Esc (

10. Configure Solaris to use DNS instead of NIS/ldap
  
#svccfg -s name-service/switch setprop config/host = astring: &#8216;(&#8220;files dns&#8221;)&#8217;
  
See http://blog.allanglesit.com/2012/05/solaris-11-dns-client-configuration-using-svccfg/

11. Enable ASLR on Solaris
  
sxadm exec -s aslr=enable /some/program
  
See http://docs.oracle.com/cd/E26502_01/html/E29031/sxadm-1m.html#REFMAN1Msxadm-1m

12. See whick package depends on library/python-2/pygobject-26
  
pkg search -l -H -o pkg.name &#8216;depend:require:library/python-2/pygobject-26&#8242;