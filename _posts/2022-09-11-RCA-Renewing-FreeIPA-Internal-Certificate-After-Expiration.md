---
title: RCA:Renewing FreeIPA Internal Certificate After Expiration
date: 2022-09-11T00:04:15+08:00
author: Zamir
layout: post

---

## English version
On September 9th, users report failed to authentication to FreeIPA.
By looking into the system status, we realize that the IPA services cannot start successfully. Most importantly, pki-tomcatd cannot start properly. With a deeper look we realized that acme.sh automatically renewed the SSL certificate and restarted FreeIPA, so we decided that should be the direct cause. With more in-depth research we realized that it's internal certificate issue.

### Incident root cause details
Internal certificates used for communication between different FreeIPA applications was expired in August. The most recent SSL certificate renewal was triggered in Sep 8th and the prior was in the very end of July. And the certificate was still valid during the renewal in July. It seems FreeIPA has a way of grace perod by only checks some certificate expiration time when restart, so that was not exporsed until acme.sh renewed the SSL certificate and restarted IPA which caused the restart faulre.

### Main issues met in recovering

1. We don't have a backup of the previous SSL certificate before each renewal. And the current SSL certification takes effect since Sep. 8th while the certificates used by internal services expired in Aug 3rd and Aug 30th respectively. In such a way, there is no single time point that we can make all the components working together.
2. When importing the server-cert.full.pem, it will check the whole CA chain for validation. It's unfortunately that DST Root CA X3 which was expired already, is still stored in our system silently with a hard-to-recognise cert name. So ipa-server-certinstall will refuse to install a certificate with error `The certificate issuer's certificate has expired.` since it thought the DST Root CA X3 is part of the CA chain and is already expired.

### Solution

#### For issue 1

I finally found a copy of old SSL certificate from a system snapshot offline on my own machine, which expires Aug 8th. So I set back the system time to early August, forcing each internal components to use port 389 + basic auth instead of 636 with certificate based authentication, and install the SSL certificate using ipa-server-certinstall. Then manually start the services following the order:
```
start-dirsrv YOUR-REALM
systemctl start pki-tomcatd@pki-tomcat.service
systemctl start krb5kdc
systemctl start kadmin
```

After that, use `kinit` to initial a valid kerberos ticket. Then use `getcert list` to check internal certificates. With the certification information listed, I tried to manual resubmit the renew request for "CN=CA Subsystem,O=YOUR-REALM" with `getcert resubmit -i <Request ID>`. By reading the system log and fixing other issues this renewal succeed. Now it's time to try renew other certificates. In this time I met an issue like the time differs with some standard time too much so LDAP refuse to do the actions. Then I decided to change the time back to today, makes sure pki-tomcatd still running, and install the current SSL certification with ipa-server-certinstall. Now issue 2 arises and I need to fix that first. Afterwards, I tried to restart FreeIPA with `ipactl restart --force` to ignore all failures and then renew other certificates. However it's still not straight forward so I decided to roll back the authentication change to use certificate authentication and port 636 again. By searching keywords form the log I noticed a suggestion about using `ipa-cert-fix  --verbose` to renew expired certificates offline. However, there is a bug with this method: The SSL certificate in our machine is not called 'Server-cert' in database, and it will fail with `ipa-cert-fix unable to fix certs no named 'Server-cert'`. The fix to the bug was applied to FreeIPA 4.9 while our production environment only has 4.6.8 and we cannot upgrade easily. So I decided to manually apply the patch by adding ipapython/directivesetter.py, updating ipaplatform/base/paths.py and partially modifying ipaserver/install/ipa_cert_fix.py according to the current environment. Note, newer FreeIPA uses mod_ssl for HTTPS while 4.6.8 uses mod_nss. With these applied, ipa-cert-fix works and fixed our issue.

#### For issue 2

* Delete the DST Root CA X3 certificate using `certutil -d dbm:/etc/httpd/alias`
* Manually removing the certificate from /etc/ipa/ca.crt
* Set back the system time and delete the expired certificate using ldapdelete

## Chinese

FreeIPA 内部子系统进行通信使用的证书分别于 8 月 3 日和 8 月 30 日部分到期。FreeIPA 的 grace period 机制是只有在重启相关服务时才会检查有效期。 acme.sh 上两次自动续期分别在 7 月底进行迁移尝试期间 和 9 月 8 日。7 月底那次恰好仍然在有效期内，所以没有发现任何问题。而 9 月 8 日 acme.sh 自动更新证书之后触发了 FreeIPA 重启，从而各组件重新检查内部各证书有效期，此时证书已经失效，遂重启失败。
修复时遇到的主要问题：
1. acme.sh 在更新证书时没有备份上一份证书，从而导致 SSL 证书有效期为 2022-09-08 之后，而内部各证书有效期为 2022-08-03 之前。因此没有任何一个时间点可以让所有应用证书均有效。这导致无法同时启动各组件进行修复。
2. 在导入  server-cert.full.pem 时会检查全部证书有效期。而已经过期的 DST Root CA X3 仍以一个不易被发现的 nickname 存在于 IPA，ipa-server-certinstall 时会误认为该证书仍是证书链必不可少的一部分，从而进行有效期检查。但由于其已经过期所以 ipa-server-certinstall 报告  The certificate issuer's certificate has expired. 

问题1 解决方案
我本地 2022 年 7 月的磁盘镜像里有一份于  2022-08-08 过期的 SSL 证书。将服务器系统时间倒回其有效期内并通过 ipa-server-certinstall 强行安装该 SSL 证书，将内部应用访问 SSL 的方式由  636 + 证书认证改为 389 + BasicAuth。之后按顺序启动 start-dirsrv YOUR-REALM， systemctl start pki-tomcatd@pki-tomcat.service，  systemctl start krb5kdc, systemctl start kadmin 并进行 kinit admin . 之后用 getcert list 获取内部证书信息，并先尝试重新提交 “CN=CA Subsystem,O=YOUR-REALM” 的续期 getcert resubmit -i <Request ID>。通过观察系统日志完成该证书的续期。之后尝试续期其他证书。此时可能会遇到某些证书续期因为本地时间与实际时间差距太大而失败。为此我将时间改为正确的时间，确保 pki-tomcatd 仍能启动，用 ipa-server-certinstall 安装最新的 SSL 证书（此时需要修复问题2），ipactl restart --force 乎略错误启动 IPA，此时可以尝试续期其他证书。但实际上我仍然没能成功续期其他证书。遂将认证方式由 389+ BasicAuth 改回 636 + 证书认证，并根据日志进行搜索，搜到了使用 ipa-cert-fix  --verbose 续期的解决方案。但该方案有一个 bug : 我本地的 SSL 证书不叫 Server-cert。此时会报错 ipa-cert-fix unable to fix certs no named 'Server-cert'。搜索发现相关修复进入了  Free IPA 4.9 但 我们线上环境为 4.6.8，无法直接升级。从而决定手动将相关修复引入。最终引入了 ipapython/directivesetter.py， 更新了 ipaplatform/base/paths.py 并部分修改了 ipaserver/install/ipa_cert_fix.py 使 ipa-cert-fix 适应当前版本（新版本 SSL 用的 mod_ssl 而 4.6.8 用的 mod_nss)。执行 ipa-cert-fix 完成修复。
问题2 解决方案
* certutil -d dbm:/etc/httpd/alias 删除 NSS 内的 DST Root CA X3 证书
* 手工删除 /etc/ipa/ca.crt 该证书
* 将本地时间倒回至内部证书有效的时间并用 ldapdelete 删除该证书


