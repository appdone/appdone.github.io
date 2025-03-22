---
title: 'TryHackMe | LazyAdmin WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [file upload, backup file, sudo]
render_with_liquid: false
media_subpath: /images/tryhackme_lazyadmin/
image:
  path: logo.webp
---

## Özet

Web uygulamasında `SweetRice` adında bir uygulama kullandığını öğreneceğiz. Daha sonra bir yedek sql dosyası ile karşılaşacağız. İçerisindeki kullanıcı bilgilerini kullanarak web uygulamasına giriş yapacak ve reverse shell dosyamızı yükleyeceğiz. Dosyayı çalıştırıp, bağlantı elde ettikten sonra ise sistemde bulunan bir dosyayı root haklarında parola gerektirmeden çalıştırabildiğimizi öğreneceğiz. Bu dosyayi incelediğimizde ise başka bir dosyayı çalıştırdığını ve çalıştırdığı dosyanın düzenleme yetkisinin bulunduğunu öğreneceğiz. İçerisine reverse shell komutumuzu girdikten sonra ilk dosyayı çalıştırarak root yetkisinde bir bağlantı alacağız.

## Keşif aşaması

### Nmap taraması

```console
$nmap -sV -sC -T4 10.10.114.17
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 ve 80 numaralı portların açık olduğunu öğreniyoruz. HTTP servisinde varsayılan Apache sayfası bulunuyor.

### 80 numaralı port'un incelenmesi

Ana sayfada varsayılan Apache sayfası bulunuyor, kaynak kodunda ise herhangi bir bilgi yok. Daha fazla bilgi edinmek adına dirsearch ile bir dosya/dizin taraması yapalım.

```console
$ dirsearch -u 10.10.114.17
...
[05:47:03] 200 -  983B  - /content/
...
```

Tarama sonucunda `content` adında bir dizin olduğunu öğreniyoruz. Bu dizindeki `index.php` sayfasının içeriği aşağıdaki şekildedir.

![](1.webp){: width="1200" height="600" }

Gördüğünüz üzere `SweetRice` adında bir yazılım kullanılıyor. `/content/changelog.txt` dosyasına baktığımızda bu yazılımın `1.5.0` sürümünü kullandığını görüyoruz. İnternette bu sürüme uygun bir exploit bulunuyor. Ancak sömürme işlemini gerçekleştirebilmek için bir kullanıcı adı ve parolaya ihtiyacımız var. dirsearch aracı ile tekrar bir arama yaptığımızda `content/inc` adında bir dizinle daha karşılaşıyoruz.

```console
$ dirsearch -u 10.10.114.17/content/
...
[05:50:29] 200 -  455B  - /content/_themes/
[05:50:52] 200 -    6KB - /content/changelog.txt
[05:51:10] 200 -  684B  - /content/images/
[05:51:10] 301 -  321B  - /content/images  ->  http://10.10.114.17/content/images/
[05:51:10] 301 -  318B  - /content/inc  ->  http://10.10.114.17/content/inc/
[05:51:10] 200 -  909B  - /content/inc/
[05:51:11] 200 -  991B  - /content/index.php/login/
[05:51:13] 200 -  535B  - /content/js/
[05:51:14] 200 -    6KB - /content/license.txt
...
```

Bu dizinin içerisinde `mysql_backup` adında bir dizin bulunuyor. Onun içerisinde ise bir sql dosyası var. Bu dosyanın içerisinde de kullanıcı bilgileri bulunuyor.

```
Kullanıcı adı: manager
Parolanın md5 karşılığı: 42f749ade7f9e195bf475f37a44cafcb
```

MD5 hash'i [hashes.com](https://hashes.com/) aracılığıyla kırdıktan sonra web sitesine giriş yapıyoruz.

## Sömürü aşaması

[bu](https://www.exploit-db.com/exploits/40716) sömürü aracını referans alarak siteye reverse shell kodumu yükleyeceğim.

![](2.webp){: width="1200" height="600" }

Siteye giriş yaptıktan sonra exploit'te belirtilen alandan `.php5` uzantılı olacak şekilde reverse shell dosyamızı yüklüyoruz.

![](3.webp){: width="1200" height="600" }

`ncat` ile belirttiğimiz port'u dinlemeye aldıktan sonra yüklediğimiz shell dosyasını çalıştırıyoruz ve böylece bir shell elde etmiş oluyoruz.

## Yetki yükseltme

### root

`sudo -l` komutunu kullandığımızda herhangi bir kullanıcının yetkisinde parola kullanmadan `perl` ile `backup.pl` adındaki bir dosyayı çalıştırabileceğimizi görüyoruz.

```console
www-data@THM-Chal:/home/itguy$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Bu dosyanın içeriği aşağıdaki şekildedir.

```perl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

`/etc/copy.sh` adında bir dosyayı çalıştırdığını görüyoruz. Bu dosyayı düzenleme yetkimiz bulunduğundan içerisinde bir reverse shell komutu giriyor ve `backup.pl` dosyasını çalıştırdıktan sonra `root` kullanıcısına ait bir shell'e geçiş yapıyoruz.

![](4.webp){: width="1200" height="600" }
