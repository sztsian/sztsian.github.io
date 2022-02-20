---
title: Using PKCS#11 Token With GPG
date: 2022-02-20T11:07:09+08:00
author: Zamir
layout: post

---
PKCS#11 is one of the popular platform-independent standard for accessing cryptographic tokens. Such tokens is widely used for various purpose in everyday life, for example USB token for your online banking or authenticating to VPN. Comparing with GPG compatible cards or tokens, PKCS#11 tokens do not have direct support by GPG, but they has a great benefit of being able to store several keys.

To use PKCS#11 tokens with GPG, we'll need to setup a gpg-agent with [gnupg-pkcs11-scd](https://github.com/alonbl/gnupg-pkcs11-scd). In this article, I'm using a [CanoKey Pigeon](https://www.canokeys.org/) with opensc-pkcs11.so library from [OpenSC](https://github.com/OpenSC/OpenSC). If you use other secure token for storing your certificate, you should consult your token provider for the library. And I assume you already have certificates on your token.

Note: This does not cover using PKCS#11 token over GPG for ssh authentication purpose. That is overkill. If you want to use PKCS#11 token for ssh authentication, take a look at documents like [this](https://docs.canokeys.org/userguide/piv/#authentication-with-ssh) without GPG in the middle.

**As of Feb, 2022, the following does not work for me with gnupg-pkcs11-scd-0.9.2-6.fc35. So I compiled gnupg-pkcs11-scd-0.10.0-1.fedorap11.fc35 myself.**

## Configure

To make this happen, make sure gnupg-pkcs11-scd is installed. Then change the gpg-agent config like the following.

```
$ cat ~/.gnupg/gpg-agent
scdaemon-program /usr/bin/gnupg-pkcs11-scd
pinentry-program /usr/bin/pinentry-gtk
```

The gnupg-pkcs11-scd daemon need a separate config file

```
providers opensc
provider-opensc-library /usr/lib64/opensc-pkcs11.so
#log-file /tmp/pkcs11log.txt
#verbose
#debug-all

```
In the config above, I am using the opensc-pkcs11.so library from OpenSC, which works with Yubikey and Canokey. However, libykpiv.so **DOES NOT WORK** for me with Yubikey. I also tested Safenet eToken 5110 and I need to use `/lib64/libeToken.so` from `SafenetAuthenticationClient-10.0.60-1.x86_64`. Newer versions does not work well for me unfortunately.

If you met some problems and need to debug, just uncomment the 3 commented lines.

Then kill any gnupg-pkcs11-scd process that is already running.

```
$ ps -ef | grep gnupg
user       13339    3375  0 11:03 ?        00:00:00 gpg-agent --homedir /home/user/.gnupg --use-standard-socket --daemon
user       13341   13339  0 11:03 ?        00:00:00 gnupg-pkcs11-scd --multi-server
user       61673   52663  0 12:20 pts/1    00:00:00 grep --color=auto gnupg
$ kill -9 13341
$ ps -ef | grep gnupg
user       13339    3375  0 11:03 ?        00:00:00 gpg-agent --homedir /home/user/.gnupg --use-standard-socket --daemon
user       61675   52663  0 12:20 pts/1    00:00:00 grep --color=auto gnupg
```

Now plugin your token, reload the gpg agent and make the daemon learn your card info.

```
$ gpg-agent --server gpg-connect-agent << EOF
> RELOADAGENT
> SCD LEARN
> EOF
OK Pleased to meet you
gpg-agent[62823]: SIGHUP received - re-reading configuration and flushing cache
gpg-agent[62823]: reading options from '/home/user/.gnupg/gpg-agent.conf'
OK
S SERIALNO D2701234567890ABCDEFA01234567891
S APPTYPE PKCS11
S KEY-FRIEDNLY AAAAAAAABBBBBBBBCCCCCCCCDDDDDDDD0000303E /CN=SSH key on SSH key
S CERTINFO 101 pkcs11:model=PKCS%2315%20emulated;token=SSH%20key;manufacturer=piv_II;serial=b78abcdef0123456;id=%01
S KEYPAIRINFO AAAAAAAABBBBBBBBCCCCCCCCDDDDDDDD0000303E pkcs11:model=PKCS%2315%20emulated;token=SSH%20key;manufacturer=piv_II;serial=b78abcdef0123456;id=%01
OK
```

Mark the hash after `KEY-FRIEDNLY` somewhere (in this example, it's `AAAAAAAABBBBBBBBCCCCCCCCDDDDDDDD0000303E`), that is the key grip used when loading the key. You should be able to see your key.

```
$ gpg2 --card-status
gpg: WARNING: server 'scdaemon' is older than us (0.10.0 < 2.3.4)                                                                                                                             
gpg: Note: Outdated servers may lack important security fixes.                                                                                                                                
gpg: Note: Use the command "gpgconf --kill all" to restart them.                                                                                                                              
Reader ...........: [none]                                                                                                                                                                    
Application ID ...: D2701234567890ABCDEFA01234567891                                                                                                                                          
Application type .: OpenPGP                                                                                                                                                                   
Version ..........: 11.50                                                                                                                                                                     
Manufacturer .....: ?                                                                                                                                                                         
Serial number ....: 4E68514A                                                                                                                                                                  
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......: 
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: forced
Key attributes ...: rsa48 rsa48 rsa48
Max. PIN lengths .: 0 0 0
PIN retry counter : 0 0 0
Signature counter : 0
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```

## Load the key to GPG

Now let's load the key by `generate key from existing`.

```
$ gpg2 --expert --full-generate-key
gpg (GnuPG) 2.3.4; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 13
Enter the keygrip: AAAAAAAABBBBBBBBCCCCCCCCDDDDDDDD0000303E

Possible actions for this RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Sign Certify Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) Y

GnuPG needs to construct a user ID to identify your key.

Real name: PKCS11 Test
Email address: test@example.com
Comment: 
You selected this USER-ID:
    "PKCS11 Test <test@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
gpg: revocation certificate stored as '/home/user/.gnupg/openpgp-revocs.d/5E5DDF886513399903AAAAAABBBBBBBB61A03A0.rev'
public and secret key created and signed.

pub   rsa2048 2022-02-20 [SCE]
      5E5DDF886513399903AAAAAABBBBBBBB61A03A0
uid                      PKCS11 Test <test@example.com>

```
The key grip is the hash after 'KEY-FRIEDNLY' in the output when you make the daemon learn your key.

There will be a prompt to type the PIN to unlock the card. Just use the PIN of the PKCS#11 token.

Now we are now good to go.

```
$ echo -n "Hello world with PKCS11" | gpg --armor --clearsign --textmode
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

Hello world with PKCS11
-----BEGIN PGP SIGNATURE-----

iQEzBAEBCAAdFiEE1ssxGDp12sfeX16wKCLPecr+3mEFAmIRxDoACgkQKCLPecr+
3mFU7wf9GYpcMDdYXq/oTJuRmAgTYuoD0JHcFxcGxPB43H4SoRHCqG9M7L/Qi1qS
/ANuf9noMabVUw659p/6GaEWWF3YVSSepxyz0iAF/q9f1QpMnn4kgwStz5/5YJCx
/tEvDee1OQu5Juru0csjZ2+T6cpwkuOPuBxQXm7RCBfzU1Fky9TplYVXkrdLpldX
O7oHxH02guybNxJ8btqQ4aknlomKNyJ94yh4pF0FdBE28R2EFWUAyjK8+gUlY989
XT6SO/vO+QqM6zHuembbUu/houUQpuB3gYK/9YJ6u5IXUBO8bregY6zCLiaHQfg6
Wd7nUfpdvd/GxWCjJMbo1fWvU163Kw==
=GBa7
-----END PGP SIGNATURE-----
```
