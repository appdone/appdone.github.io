---
title: 'PicoCTF 2023 | FindMe WriteUp'
author: appdone
categories: [picoCTF 2023 Challenges]
render_with_liquid: false
media_subpath: /images/picoctf_findme/
image:
  path: 1.webp
---

Bu yazıda, picoCTF platformunda yer alan "FindMe" isimli meydan okumayı çözeceğiz. Verilen kullanıcı bilgileriyle giriş sayfasını test etmemiz isteniyor. Giriş yaptıktan sonra bir yerlere yönlendiriyor, daha sonra ise kullanıcı paneline yönlendiriyor.

![dashboard](2.webp){: width="1000" height="500" }

Yönlendirmeleri takip etmek için geliştirici araçlarını kullanabiliriz. Geliştirici ayarlarını açtıktan sonra network sekmesini açın ve "Preserve log" yazan kutucuğu işaretleyin. Böylece siteye giriş yapmaya çalıştığımızda yönlendirilen bütün URL adreslerini gösterecek.

![dashboard](3.webp){: width="1000" height="500" }

Gördüğünüz üzere "id" parametrelerinde base64 benzeri birer veri bulunuyor. Bunları birleştirip CyberChef sitesinde çözdükten sonra bayrağa ulaşıyoruz.
