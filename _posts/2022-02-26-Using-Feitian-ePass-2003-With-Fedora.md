---
title: Using Feitian ePass 2003 With PKCS#15 on Fedora
date: 2022-02-26T15:22:25+08:00
author: Zamir
layout: post

---

[Fetian ePass 2003](https://web.archive.org/web/20220226072631/https://www.ftsafe.com/Products/PKI/Standard/Specification) is a USB token targeting standard PKI user cases. It supports RSA up to 2048. It has a CCID interface and is supported by the [open source CCID](https://salsa.debian.org/rousseau/CCID/) driver. So it's much easier to use in Linux. In this article I'm taking a note on how I use the card with opensc and openssl on Fedora.

Firstly, erase the token and initialize it with PKCS#15.
```
$ pkcs15-init -E
Using reader with a card: Feitian ePass2003 00 00

$ pkcs15-init --create-pkcs15 --profile pkcs15+onepin --label “epass2003”
Using reader with a card: Feitian ePass2003 00 00
New User PIN.
Please enter User PIN: 
Please type again to verify: 
Unblock Code for New User PIN (Optional - press return for no PIN).
Please enter User unblocking PIN (PUK): 
Please type again to verify: 
```
Generate a keypare with RSA2048. Note, without `--auth-id`, pkcs15-init will fail. And specify a label for the keypair will make it easier for openssl later. In this case, I label the keypair as 'key2048'.

```
$ pkcs15-init --auth-id 1 --generate-key rsa/2048 --key-usage sign,decrypt --label "key2048"
Using reader with a card: Feitian ePass2003 00 00
User PIN [User PIN] required.
Please enter User PIN [User PIN]: 
```

Checking the token, we can see the keys has been generated.

```
$ pkcs15-tool --list-keys --list-public-keys
Using reader with a card: Feitian ePass2003 00 00
Private RSA Key [key2048]
        Object Flags   : [0x03], private, modifiable
        Usage          : [0x2E], decrypt, sign, signRecover, unwrap
        Access Flags   : [0x1D], sensitive, alwaysSensitive, neverExtract, local
        Algo_refs      : 0
        ModLength      : 2048
        Key ref        : 0 (0x00)
        Native         : yes
        Path           : 3f0050152900
        Auth ID        : 01
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3
        MD:guid        : 3b1e1409-9f2f-eed4-a372-406e8b318ff7                                                                                                                                 
                                                                                                                                                                                              
Public RSA Key [key2048]                                                                                                                                                                      
        Object Flags   : [0x02], modifiable                                                                                                                                                   
        Usage          : [0xD1], encrypt, wrap, verify, verifyRecover                                                                                                                         
        Access Flags   : [0x00]                                                                                                                                                               
        ModLength      : 2048                                                                                                                                                                 
        Key ref        : 0 (0x00)                                                                                                                                                             
        Native         : no
        Path           : 3f0050153000
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3
```

The token is now ready to work with ssh. You can check the publick key using pkcs15-tool,

```
$ pkcs15-tool --read-ssh-key e484031ff1c68907080c67630b79d569c7e0dde3
Using reader with a card: Feitian ePass2003 00 00
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCe2jkdNslyxab93BgssLTqRjvJZloAwGev/97bKqhsT5k0rSyn/98Cxk0oT6L84n2wTZ+xAENLJ9M5DJLBVMz2bACVEXqNe0HAAyrDwjfvF5cqKXa4sPkUrb+yq3RrVry9OXMm7I/mHeRUSOFkwvKn1SR4kpEVvjpm896W/KNFOM5JEqK4/A4m28gYOKNcKBUK/gShpfxvbdBszBF2jkzni1a+Btil0oLVv/ha/yOxU3CKzhs8EM42LM449HrXK5uaX/2YDz/wNUDlgR7YaNXkS2E7wauEOfI6ukF9AfUTyU+/f6x32KoN7fbiZLvgicwD8qNeBb7UvVRcb2Jyisxv key2048
```

Or using ssh with opensc-pkcs11.so library from opensc.

```
$ ssh-keygen -D /usr/lib64/opensc-pkcs11.so -e
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCe2jkdNslyxab93BgssLTqRjvJZloAwGev/97bKqhsT5k0rSyn/98Cxk0oT6L84n2wTZ+xAENLJ9M5DJLBVMz2bACVEXqNe0HAAyrDwjfvF5cqKXa4sPkUrb+yq3RrVry9OXMm7I/mHeRUSOFkwvKn1SR4kpEVvjpm896W/KNFOM5JEqK4/A4m28gYOKNcKBUK/gShpfxvbdBszBF2jkzni1a+Btil0oLVv/ha/yOxU3CKzhs8EM42LM449HrXK5uaX/2YDz/wNUDlgR7YaNXkS2E7wauEOfI6ukF9AfUTyU+/f6x32KoN7fbiZLvgicwD8qNeBb7UvVRcb2Jyisxv key2048 pkcs11:id=%E4%84%03%1F%F1%C6%89%07%08%0C%67%63%0B%79%D5%69%C7%E0%DD%E3;object=key2048;token=%E2%80%9Cepass2003%E2%80%9D%20(User%20PIN);manufacturer=EnterSafe?module-path=/usr/lib64/opensc-pkcs11.so

```

However, there are no certificates yet.

```
$ pkcs15-tool --list-certificates
Using reader with a card: Feitian ePass2003 00 00
```

Without a certificate, the token cannot be used for web authentication or to use with gnupg-pkcs11-scd. A certificate can be created using openssl. Note, the label of the keypair is needed.

```
$ openssl req -engine pkcs11 -new -key "pkcs11:object=key2048"  -keyform engine -out epass2003cert.pem -days 3650 -outform pem -x509 -utf8
engine "pkcs11" set.
Enter PKCS#11 token PIN for “epass2003” (User PIN):
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:Test 2003
Email Address []:test@example.com
$ 
```

The last thing before we can use is to import the certificate into the token.

```
$ pkcs15-init --store-certificate epass2003cert.pem --verify-pin
Using reader with a card: Feitian ePass2003 00 00
User PIN required.
Please enter User PIN [User PIN]: 
```

Now, the certificate is in the token.
```
$ pkcs15-tool --list-certificates
Using reader with a card: Feitian ePass2003 00 00
X.509 Certificate [Certificate]
        Object Flags   : [0x02], modifiable
        Authority      : no
        Path           : 3f0050153100
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3
        Encoded serial : 02 14 1C1EFCCACC9A91806818C1B5F0BA020160A5251A


$ pkcs15-tool --dump
Using reader with a card: Feitian ePass2003 00 00
PKCS#15 Card [“epass2003”]:
        Version        : 0
        Serial number  : 2ade2a67000a801b
        Manufacturer ID: EnterSafe
        Last update    : 20220226031311Z
        Flags          : EID compliant


PIN [User PIN]
        Object Flags   : [0x03], private, modifiable
        ID             : 01
        Flags          : [0x32], local, initialized, needs-padding
        Length         : min_len:4, max_len:16, stored_len:16
        Pad char       : 0x00
        Reference      : 1 (0x01)
        Type           : ascii-numeric
        Path           : 3f005015

Private RSA Key [Certificate]
        Object Flags   : [0x03], private, modifiable
        Usage          : [0x2E], decrypt, sign, signRecover, unwrap
        Access Flags   : [0x1D], sensitive, alwaysSensitive, neverExtract, local
        Algo_refs      : 0
        ModLength      : 2048
        Key ref        : 0 (0x00)
        Native         : yes
        Path           : 3f0050152900
        Auth ID        : 01
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3
        MD:guid        : 3b1e1409-9f2f-eed4-a372-406e8b318ff7

Public RSA Key [key2048]
        Object Flags   : [0x02], modifiable
        Usage          : [0xD1], encrypt, wrap, verify, verifyRecover
        Access Flags   : [0x00]
        ModLength      : 2048
        Key ref        : 0 (0x00)
        Native         : no
        Path           : 3f0050153000
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3

X.509 Certificate [Certificate]
        Object Flags   : [0x02], modifiable
        Authority      : no
        Path           : 3f0050153100
        ID             : e484031ff1c68907080c67630b79d569c7e0dde3
        Encoded serial : 02 14 1C1EFCCACC9A91806818C1B5F0BA020160A5251A
```

And it should work with gnupg-pkcs11-scd + gpg now

```
$ gpg-agent --server gpg-connect-agent << EOF
> RELOADAGENT
> SCD LEARN
> EOF
OK Pleased to meet you
gpg-agent[204854]: SIGHUP received - re-reading configuration and flushing cache
gpg-agent[204854]: reading options from '/home/user/.gnupg/gpg-agent.conf'
OK
S SERIALNO D276000124011150313131FAE6891111
S APPTYPE PKCS11
S KEY-FRIEDNLY E0684F1AFBB4FB5B75EC12C737C28F2F972DF63B /C=CN/L=Default City/O=Default Company Ltd/CN=Test 2003/emailAddress=test@example.com on “epass2003” (User PIN)
S CERTINFO 101 pkcs11:model=PKCS%2315;token=%e2%80%9cepass2003%e2%80%9d%20%28User%20PIN%29;manufacturer=EnterSafe;serial=2ade2a67000a801b;id=%e4%84%03%1f%f1%c6%89%07%08%0cgc%0by%d5i%c7%e0%dd%e3
S KEYPAIRINFO E0684F1AFBB4FB5B75EC12C737C28F2F972DF63B pkcs11:model=PKCS%2315;token=%e2%80%9cepass2003%e2%80%9d%20%28User%20PIN%29;manufacturer=EnterSafe;serial=2ade2a67000a801b;id=%e4%84%03%1f%f1%c6%89%07%08%0cgc%0by%d5i%c7%e0%dd%e3
OK

$ gpg2 --card-status
gpg: WARNING: server 'scdaemon' is older than us (0.10.0 < 2.3.4)
gpg: Note: Outdated servers may lack important security fixes.
gpg: Note: Use the command "gpgconf --kill all" to restart them.
Reader ...........: [none]
Application ID ...: D276000124011150313131FAE6891111
Application type .: OpenPGP
Version ..........: 11.50
Manufacturer .....: ?
Serial number ....: 31FAE689
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

Some useful references are

* Feitian's comment about the workflow to use ePass2003 with opensc - https://github.com/OpenSC/OpenSC/issues/2424#issuecomment-954563982
* A blog on how to make this work on Windows - https://zerowidthjoiner.net/2019/01/12/using-ssh-public-key-authentication-with-a-smart-card
* https://zerowidthjoiner.net/uploads/2019/01-12/feitian-epass-nfc.pdf
* http://cedric.dufour.name/blah/IT/SmartCardsHowto.html
* https://www.mayrhofer.eu.org/post/aladdin-etoken-under-linux/
* https://blog.runtux.com/posts/2009/12/05/150/

