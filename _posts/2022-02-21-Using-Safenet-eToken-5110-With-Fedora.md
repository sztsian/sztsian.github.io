---
title: Using Safenet eToken 5110 With Fedora
date: 2022-02-21T22:44:15+08:00
author: Zamir
layout: post

---

[Safenet eToken 5110](https://web.archive.org/web/20220221145346/https://cpl.thalesgroup.com/access-management/authenticators/pki-usb-authentication/etoken-5110-usb-token) is a smart card based USB authenticator focuse on certificate based user cases. It supports RSA2048. In this article I'm taking a note on how I use the card with `pkcs11-tool` and `openssl` on Fedora. This is based on [Using Tokens in Ubuntu with PGP](https://craftware.xyz/securitybricks/2017/07/17/using-tokens-in-Ubuntu-with-pgp.html) with information slightly modified to fulfill my environment.

The pkcs11-tool module I am using is `/usr/lib64/libeToken.so` from [SafenetAuthenticationClient-10.0.60](https://knowledge.digicert.com/generalinformation/INFO1982.html).

Initial the token, set SO PIN and PIN.

```
$ pkcs11-tool --module /usr/lib64/libeToken.so --init-token --label my5110
Using slot 0 with a present token (0x0)
Please enter the new SO PIN: 
Please enter the new SO PIN (again): 
Token successfully initialized
$ pkcs11-tool --module /usr/lib64/libeToken.so --init-pin --login
Using slot 0 with a present token (0x0)
Logging in to "my5110".
Please enter SO PIN: 
Please enter the new PIN: 
Please enter the new PIN again: 
User PIN successfully initialized
```
Generate a keypare with RSA2048.

```
$ pkcs11-tool --module /usr/lib64/libeToken.so --login --keypairgen --key-type RSA:2048 --id 1 --label "test@example.com"
Using slot 0 with a present token (0x0)
Logging in to "my5110".
Please enter User PIN: 
Key pair generated:
Private Key Object; RSA 
  label:      test@example.com
  ID:         01
  Usage:      decrypt, sign, unwrap
  Access:     sensitive, always sensitive, never extractable, local                                                                                                                           
Public Key Object; RSA 2048 bits                                                                                                                                                              
  label:      test@example.com                                                                                                                                                                
  ID:         01                                                                                                                                                                              
  Usage:      encrypt, verify, wrap                                                                                                                                                           
  Access:     local
```

Now the keys are generated. By checking the objects we can see 2 there, one for public key and 1 for private key.

```
$ pkcs11-tool --module /usr/lib64/libeToken.so --login --list-objects
Using slot 0 with a present token (0x0)                                                                                                                                                       
Logging in to "my5110".                                                                                                                                                                       
Please enter User PIN: 
Private Key Object; RSA 
  label:      test@example.com
  ID:         01
  Usage:      decrypt, sign, unwrap
  Access:     sensitive, always sensitive, never extractable, local
Public Key Object; RSA 2048 bits
  label:      test@example.com
  ID:         01
  Usage:      encrypt, verify, wrap
  Access:     local
```
Now we can create a x509 certificate. Here we use OpenSSL with an engine. The path of the engine is available by

```
$ rpm -ql openssl-pkcs11 | grep engine                                                                                                                                             
/usr/lib64/engines-1.1/libpkcs11.so
/usr/lib64/engines-1.1/pkcs11.so
/usr/lib/engines-1.1/libpkcs11.so
/usr/lib/engines-1.1/pkcs11.so
```

Use the 64bit pkcs11.so above in the following commands.

```
$ openssl 
OpenSSL> engine dynamic -pre SO_PATH:/usr/lib64/engines-1.1/pkcs11.so -pre ID:pkcs11 -pre LIST_ADD:1 -pre LOAD -pre MODULE_PATH:libeToken.so
(dynamic) Dynamic engine loading support
[Success]: SO_PATH:/usr/lib64/engines-1.1/pkcs11.so
[Success]: ID:pkcs11
[Success]: LIST_ADD:1
[Success]: LOAD
[Success]: MODULE_PATH:libeToken.so
Loaded: (pkcs11) pkcs11 engine
OpenSSL> req -engine pkcs11 -new -key slot_0-id_01 -keyform engine -x509 -out my.pem -text
engine "pkcs11" set.
Enter PKCS#11 token PIN for my5110:
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
Common Name (eg, your name or your server's hostname) []:Test CN 
Email Address []:
OpenSSL> x509 -in my.pem -out my.der -outform der
OpenSSL> 
$ 
```

The last thing before we can use is to import the certificate into the token.

```
$ pkcs11-tool --module /usr/lib64/libeToken.so --login --write-object my.der --type cert --id 01 --label "test@example.com"
Using slot 0 with a present token (0x0)
Logging in to "my5110".
Please enter User PIN: 
Created certificate:
Certificate Object; type = X.509 cert
  label:      test@example.com
  subject:    DN: C=CN, L=Default City, O=Default Company Ltd, CN=Test CN
  ID:         01
```

For using with gnupg-pkcs11-scd, my config looks like

```
$ grep -v '^#' .gnupg/gnupg-pkcs11-scd.conf 
providers safenet
provider-safenet-library /lib64/libeToken.so
log-file /tmp/pkcs11log.txt
verbose
debug-all
```

Some useful references are

* https://eideasy.com/pkcs-11-rsa-signature-with-opensc-and-gemalto-etoken-5110/
* https://craftware.xyz/securitybricks/2017/07/17/using-tokens-in-Ubuntu-with-pgp.html
