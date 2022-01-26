---
title: Using PIV Smartcard in FreeIPA
date: 2022-01-26T22:07:19+08:00
author: Zamir
layout: post

---
Personal Identity Verification (PIV) is a standard proposed by the US government for identification and now is supported by various smart cards and USB secure tokens. FreeIPA have supported authenticating with PIV certificate but is not enabled by default. In this article, I'll cover how to use PIV authenticate from user perspective with an existing FreeIPA that enabled the corresponding support.

In this article, I'm using a [CanoKey Pigeon](https://www.canokeys.org/) with the `ykman` command from Yubico. It should work exactly the same with Yubikey (just omit the `-r canokeys` from all my following commands). If you use other secure token for storing your certificate, you should consult your token provider.

If you haven't initialized PIV on your token, you should firstly initial the Cardholder Unique Identifier (CHUID) and Cardholder Capability Container (CCC).

```
ykman -r canokeys piv objects generate chuid
ykman -r canokeys piv objects generate ccc
```

Then generate your private key on the card. I'm using the slot 9a, which is the slot for authentication.

```
ykman -r canokeys piv keys generate 9a 9a-pub-key.pem
```

Then generate the Certificate Signing Request (CSR).

```
ykman -r canokeys piv certificates request -s 'CN=user,O=EXAMPLE.COM' 9a ./9a-pub-key.pem 9a-csr.pem
```

Now login to your FreeIPA server, you'll see your profile page. Click the drop-down button 'Actions' then you'll see 'New Certificate'. Click it.

Choose the profile ID for your certificate. Usually you'll need 'IECUserRoles' by default. Then Copy and paste the content of the CSR file `9a-csr.pem` you just generated to the big text area below. Click 'Issue'. Then you'll see the certificate summary generated for your user. Close the box and you'll then see there is a certificate showing up in 'Certificates' area. Click the 'Action' of that certificate and download the certificate. I'm saving it as '9a-issued-cert.pem'.

Back to the command line, let's import the issued certificate to your token

```
ykman -r canokeys piv certificates import 9a ./9a-issued-cert.pem
```

Now everything has been set. You can try by clicking the "Login using personal certificate" hyperlink in the FreeIPA login page.

