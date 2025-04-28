---
title: 'VulnHub | Who Wants To Be King WriteUp'
author: appdone
categories: [VulnHub Challenges]
render_with_liquid: false
---

## Özet

Game of Thrones dizisi temalı bu meydan okumada, belli başlı dosyaları inceleyerek kullanıcılar arasında geçiş yapacağız.

### Makine hakkında ek bilgiler

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sVC -T4 192.168.111.13
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7f:55:2d:63:a8:86:4f:90:1f:05:3c:c9:9f:40:b3:f2 (RSA)
|   256 e9:71:11:ed:17:fa:48:06:a7:6b:5b:b6:0e:1b:11:b8 (ECDSA)
|_  256 db:74:42:c4:37:c3:ae:a0:5c:30:26:cb:1a:ef:76:52 (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Index of /
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 31K   2020-12-01 11:23  skeylogger
|_
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucuna göz attığımızda HTTP servisi üzerinde `skeylogger` isimli bir dosyanın listelendiğini görüyoruz. Bu dosyayı indirip, inceleyelim.

### skeylogger isimli dosyanın incelenmesi

```console
$ file skeylogger 
skeylogger: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ba22a62cfb23e5f98841e89718b9d3f5e76bdf94, for GNU/Linux 3.2.0, with debug_info, not stripped
```

Elimizdeki dosyanın bir elf dosyası yani linux sistemlerinde çalıştırılabilir bir dosya olduğunu biliyoruz. Dosyanın içerisindeki okunabilir metinleri incelediğimizde base64 ile kodlanmış gibi görünen bir metin ve dosyanın indirildiği dizin ile karşılaşıyoruz.

```console
$ strings skeylogger
...
Could not determine keyboard device file
ZHJhY2FyeXMK
Usage: skeylogger [OPTION]
Logs pressed keys
  -h, --help		Displays this help message
  -v, --version		Displays version information
  -l, --logfile		Path to the logfile
  -d, --device		Path to device file (/dev/input/eventX)
Simple Key Logger version 0.0.1
grep -E 'Handlers|EV' /proc/bus/input/devices |grep -B1 120013 |grep -Eo event[0-9]+ |tr '\n' '\0'
...
/home/sunita/Descargas/simple-key-logger-master
...
```

Dosyanın indirildiği dizin yolundan yola çıkarak sistemde `sunita` adında bir kullanıcı olduğunu düşünebiliriz. Base64 ile kodlanmış veriyi çözdüğümüzde ise `Game of Thrones` dizisinden bir ejderhanın adı ile karşılaşıyoruz. Diziyi izlemedim ama meşhur sahnelerinden birinde bu adı duymuştum.


```console
$ echo "ZHJhY2FyeXMK" | base64 -d
dracarys
```

Bu isim, `sunita` adlı kullanıcının parolası olabilir diye düşündüm ama denediğimde giriş yapamadım. Ghidra ile dosyanın ne işe yaradığını anlamaya/çözmeye çalıştım. Kendisi, dosyanın root haklarında çalıştırılıp çalıştırılmadığını kontrol ediyor ve hak sağlandığı durumda, bulduğumuz base64 kodlaması adında bir dosya oluşturuyor. Oluşturduğu dosyanın içerisinde önemli bir bilgi yok.

`dracarys` kelimesini googleda aradığımda bu kelimenin `Valyrian` dilinde `ejderha ateşi` anlamına geldiğini öğrendim. Ejderhaların ateş püskürtmesi istenildiğinde bu kelime kullanılıyormuş. Dizide bu kelimeyi `daenerys targaryen` karakteri kullanıyormuş. İsminden yola çıkarak SSH servisine, `dracarys` parolasını kullanarak giriş yapabiliyoruz.

## Yetki yükseltme

### root

Sistemde `sunita` adında bir kullanıcı bulunmuyor, kandırılmışız. Home dizininde `secret` adında bir dosya ile karşılaşıyoruz. İçerisinde aşağıdaki mesaj bulunuyor.

```console
daenerys@osboxes:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  secret  Templates  Videos
daenerys@osboxes:~$ cat secret 
find home, pls
```

`sudo -l` komutunu kullandığımızda ise root haklarıyla birkaç dosyayı çalıştırma yetkimizin olduğunu öğreniyoruz.

```console
daenerys@osboxes:/var/www/html$ sudo -l
Matching Defaults entries for daenerys on osboxes:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, pwfeedback

User daenerys may run the following commands on osboxes:
    (root) NOPASSWD: /usr/bin/mint-refresh-cache
    (root) NOPASSWD: /usr/lib/linuxmint/mintUpdate/synaptic-workaround.py
    (root) NOPASSWD: /usr/lib/linuxmint/mintUpdate/dpkg_lock_check.sh
```

Tabi bu dosyaların hiçbiri sistemde bulunmuyor. Home dizinindeki .bash_history dosyasını kontrol ettiğimizde ise `.local/share` dizinindeki `djkdsnkjdsn` isimli dosyayı sildiğini görüyoruz.


```console
daenerys@osboxes:~$ cat .bash_history 
cd .local/
ls
cd share/
ls
rm djkdsnkjdsn 
cd
ls
ls -la
rm .bash_history 
...
```

Bulunduğu dizine gittiğimizde `daenerys.zip` dosyası ile karşılaşıyoruz. İçerisinde silindiğini öğrendiğimiz dosya bulunuyor. Bu dosyanın içerisinde ise başka bir dizindeki dosya referans verilmiş. 


```console
daenerys@osboxes:~/.local/share$ unzip daenerys.zip 
Archive:  daenerys.zip
 extracting: djkdsnkjdsn             
daenerys@osboxes:~/.local/share$ ls
daenerys.zip  djkdsnkjdsn  evolution  flatpak  gnote  nano
daenerys@osboxes:~/.local/share$ cat djkdsnkjdsn 
/usr/share/sounds/note.txt
daenerys@osboxes:~/.local/share$ cat /usr/share/sounds/note.txt
I'm khal.....
```

Dosyanın içerisindeki `khal...` ismini GOT dizisinden yola çıkarak tamamladıktan sonra parola olarak kullanıyor ve root kullanıcısına geçiş yapıyoruz.

```console
daenerys@osboxes:~/.local/share$ su root
Password: 
root@osboxes:/home/daenerys/.local/share# cd 
root@osboxes:~# ls -la
...
-rw-r--r--  1 root root  106 Dec  1  2020 nice.txt
...
root@osboxes:~# cat nice.txt 
¡Congratulation!
You have a good day!
aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj1nTjhZRjBZZmJFawo=
```
