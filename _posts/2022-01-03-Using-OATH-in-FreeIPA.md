---
title: Using OATH in FreeIPA
date: 2022-01-03T22:17:59+08:00
author: Zamir
layout: post

---
OATH has been the choice of Two-factor authentication (2FA) for many companies and websites. And it's also exists in FreeIPA.

As a user, it's pretty straight forward to enable OATH token in FreeIPA on you own given your organization has enabled the choice for you.

Firstly, login into FreeIPA with your own account. Click your own users from the 'Active users' table.

Click the drop-down button 'Action' then you'll see 'Add OTP Token'. Click it.

Then it will ask you to choose whether it's TOTP or HOTP. Choose on your own preference. You can file in something in `Description` to make it easier for you to tell your tokens apart. Then click 'Add'.

On the next screen, you'll be offered a QE code.

If you want to use a mobile app, then just scan the QR code with the mobile app you want to use. Done.

However, if you want to store the OATH token in a hardware token, like CanoKey or Yubikey, you'd better click 'Show configuration uri'. Then configure it using the configuration tool. In my case, I'm using `ykman` with a [CanoKey Pigeon](https://docs.canokeys.org/userguide/oath/).

```
ykman -r "Canokeys" oath accounts uri "otpauth://Hotp/username@EXAMPLE.COM:12345678-90ab-cdef-1234-567890abcdef?digits=6&secret=SOMESECRET&period=30&algorithm=SHA1&issuer=username%40EXAMPLE.COM"
```

Now everything has been set. Go on and configure your hardware token as needed, and you are good to go.

If you are using HOTP and your token now has a big skew now, you can go to the login page of FreeIPA, click 'Sync OTP Token' and sync your token there.

Easy, isn't it?
