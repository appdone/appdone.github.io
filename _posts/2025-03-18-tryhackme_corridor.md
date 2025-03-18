---
title: 'TryHackMe | Corridor WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [idor]
render_with_liquid: false
media_subpath: /images/tryhackme_corridor/
image:
  path: 1.webp
---

Kendimizi garip bir koridorun ortasında bulduk ve buradan çıkmak için doğru kapıyı bulmamız gerekiyor. Her kapının md5 ile hashlenmiş bir numarası bulunuyor.

![](2.webp){: width="1200" height="600" }

Koridorda gördüğümüz bütün kapıların ardında boş bir oda var.

![](3.webp){: width="1200" height="600" }

Bu durumda koridorda gizlenmiş bir kapıyı bulmamız gerekiyor. Oda numaralarının nasıl oluşturulduğunu biliyoruz. 1'den 13'e kadar oda numaraları bulunuyor. Şahsen ben gizli bir oda yapsaydım, numarasını 0 veya 14 koyardım. Bu numaraları md5 hash’e dönüştürüp kontrol edelim.

![](4.webp){: width="1200" height="600" }

Sıfır numaralı odayı hash’leyip görüntülediğimizde gizli odayı bulmuş oluyoruz.
