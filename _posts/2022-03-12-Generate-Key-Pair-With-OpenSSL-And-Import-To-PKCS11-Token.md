---
title: Generate Key Pair With OpenSSL And Import To PKCS#11 Token
date: 2022-03-12T23:04:53+08:00
author: Zamir
layout: post

---

As I'm playing with PKCS#11 token a lot recently, I'm now thinking about generating all essential data off the card and then importing. This is less secure but makes backup possible. So I tried with OpenSSL to generate everything needed.

Let's start by generate an RSA2048 key pair with openssl.
```
$ openssl genrsa -aes256 -out testkey.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..................................................................................+++++
...............................................+++++
e is 65537 (0x010001)
Enter pass phrase for testkey.key:
Verifying - Enter pass phrase for testkey.key:
```

The following command can be used to inspect the private key.

```
$ openssl rsa -text -in testkey.key
```

To export the pub key, the following command is needed.
```
$ openssl rsa -in testkey.key -pubout -out testkey-public.key
Enter pass phrase for testkey.key:
writing RSA key
```

Then everything is prepared for certificate. For generating a CSR to send to public CA, follow the command below.

```
$  openssl req -new -key testkey.key -out testkey.csr
Enter pass phrase for testkey.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:Test Group
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server's hostname) []:Test-Group
Email Address []:test@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:.
An optional company name []:
```

The CSR details can be inspected with `openssl req -text -in testkey.csr -noout`.

To generate a self-signed certificate, the following command does the job.
```
$ openssl x509 -req -days 3654 -in testkey.csr -signkey testkey.key -out testkey.crt
Signature ok
subject=C = CN, O = Test Group, CN = Test-Group, emailAddress = test@example.com
Getting Private key
Enter pass phrase for testkey.key:
```

For generating a self-signed certificate, the above two steps can be merged as the following.

```
$ openssl req -new -x509 -days 3654 -key testkey.key -out testkey.crt
Enter pass phrase for testkey.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:Test Group 
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server's hostname) []:Test-Group
Email Address []:test@example.com
```

Depending on the token for storing the certificate, the private key and certificate might need to be converted.

To convert into a pfx file containing both the key pair and the certificate, use the following commands.

```
$ openssl pkcs12 -export -in testkey.crt -inkey testkey.key -out testkey.pfx
Enter pass phrase for testkey.key:
Enter Export Password:
Verifying - Enter Export Password:
```

To convert into DER format, the following commands helps.

```
$ openssl rsa -in ./testkey.key -outform DER -out testkey-key.der
Enter pass phrase for ./testkey.key:
writing RSA key
$ openssl x509 -outform DER -in testkey.crt -out testkey-crt.der
```

Then the keys and certificate can be imported to a PKCS#11 token, for example, using pkcs11-tool like below.
```
$ pkcs11-tool --login --write-object ~/tmp/testkey-key.der --type privkey --id 1
$ pkcs11-tool --login --write-object ~/tmp/testkey-crt.der --type cert --id 1
$ pkcs11-tool --login --write-object ~/tmp/testkey-public.key --type pubkey --id 1

```

One interesting finding: The gnupg-pkcs11-scd daemon can detect a key in token which the private key and certificate exists but not the public key. However, it does not work if only the private key and public key are in the token without the certificate.

References:
* [OpenSSL Cookbook](https://www.feistyduck.com/books/openssl-cookbook/)
* [Importing key and certificate using pkcs11-tool and getting it from java application](https://wiki.onap.org/display/DW/Importing+key+and+certificate+using+pkcs11-tool+and+getting+it+from+java+application).
* [How to convert a certificate into the appropriate format](https://knowledge.digicert.com/solution/SO26449.html)

