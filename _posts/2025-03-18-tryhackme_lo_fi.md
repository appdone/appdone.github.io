---
title: 'TryHackMe | Lo-Fi WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [lfi]
render_with_liquid: false
media_subpath: /images/tryhackme_lo_fi/
image:
  path: logo.webp
---

Bu yazıda, TryHackMe platformunda yer alan “Lo-Fi” adlı meydan okumayı çözeceğiz. Meydan okumanın amacı, web uygulamasında bulunan “Local File Inclusion” zafiyetinden yararlanarak sistemin kök dizinindeki bayrağı ele geçirmektir.

![](1.webp){: width="1200" height="600" }

Url yapısını incelediğimizde index.php dosyasının page parametresi ile relax.php dosyasını sayfaya dahil ettiğini görüyoruz. Bu dosya yerine flag.txt dosyasını girdiğimizde aşağıdaki hata ile karşılaşıyoruz.

![](2.webp){: width="1200" height="600" }

İlk başta ‘/’ ve ‘.’karakterlerini engellediğini düşünmüştüm ama tek başına parametreye eklediğimde hata vermiyordu. Bu yüzden bayrağı doğrudan kök dizinden almak yerine “../” ifadesini ekleye ekleye kök dizine kadar ulaşıp bayrağa erişmeye çalıştım.

Buradaki “../” ifadeleri bir alt satır anlamına geliyor. “../../../flag.txt” şeklinde girdiğimde ise 3 alt satırdaki yani kök dizindeki flag dosyasını sayfaya dahil ediyor.

![](3.webp){: width="1200" height="600" }
