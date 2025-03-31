---
title: 'HackTheBox Challenges | SpookyPass WriteUp'
author: appdone
categories: [HackTheBox Challenges]
tags: [reverse engineering]
render_with_liquid: false
media_subpath: /images/hackthebox_challenges_spookypass/
image:
  path: logo.webp
---

Kasabadaki en havalı hayaletler bir perili ev partisine gidiyor. Bu büyük partiye katılabilmemiz için verilen dosyayı analiz etmemiz ve sonunucda dosya üzerinden davet kodunu almamız gerekiyor. Öncelikle dosyayı yetkilendirip, çalıştıralım ve bizden ne istediğini görelim.

```console
$ chmod +x pass
$ ./pass 
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password: appdone
You're not a real ghost; clear off!
```

Bir parola istiyor, geçersiz parola girdiğimizde bizim hayalet olmadığımızı anlıyor ve bizi kapı dışarı ediyor. Ghidra aracını kullanarak dosyayı decompile ettikten sonra `main` fonksiyonunu görüntüleyelim.

![](1.webp){: width="1200" height="600" }

Gördüğünüz üzere girilen parolanın elindeki parola ile eşleşip eşleşmediğini kontrol ediyor ve eşleşme durumunda for döngüsünü kullanarak bayrağı ekrana yazdırıyor. Programı tekrar başlatalım ve gördüğümüz parolayı girelim.

```console
$./pass 
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password: s3cr3t_***_gh0ul5
Welcome inside!
HTB{***}
```
