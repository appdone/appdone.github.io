---
title: 'picoCTF 2023 | Special WriteUp'
author: 'appdone'
categories: [picoCTF 2023 Challenges]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "Special" isimli meydan okumayı çözeceğiz. Makineyi başlattıktan sonra SSH servisine bağlanıp, komutlar çalıştırmayı deniyoruz. Ancak girilen komutların ismini cismini değiştiren bir shell kullandığımız için girdiğimiz komutlar çalışmıyor. Shell değiştirmeye çalıştığımızda ise buna izin vermediğini görüyoruz.

```console
Special$ ls
Is 
sh: 1: Is: not found
Special$ pwd
Pod 
sh: 1: Pod: not found
Special$ bash
Why go back to an inferior shell?
Special$ 
```

Biraz kurcaladıktan sonra ". & id" şeklinde komut çalıştırmayı başardım. Tabi "ls" gibi komutları yine çalıştıramıyorum. ". & echo *" şeklinde bir komut kullandığımda bulunduğum dizinde "blargh" adında bir dizin olduğunu ve içerisinde bayrağın bulunduğunu öğrendim. Sonrada bayrağı olduğu yerden çekip çıkardım.

```console
Special$ . & id
. & id 
uid=1000(ctf-player) gid=1000(ctf-player) groups=1000(ctf-player)
Special$ . & echo *
. & echo * 
blargh
Special$ . & echo blargh/*
. & echo blargh/* 
blargh/flag.txt
Special$ . & cat blargh/flag.txt
. & cat blargh/flag.txt 
picoCTF{***}
Special$ 
```
