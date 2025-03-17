---
title: 'picoCTF 2022 | Power Cookie WriteUp'
author: appdone
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
media_subpath: /images/picoctf_power_cookie/
image:
  path: 1.webp
---

Bu yazıda, picoCTF platformunda yer alan "Power Cookie" isimli meydan okumayı çözeceğiz.

![](1.webp){: width="1000" height="500" }

Ana sayfada misafir olarak devam et yazısı bulunan bir buton var. Tıkladıktan sonra hizmet veremeyeceklerini söyleyen bir yazı ile karşılaşıyoruz. Meydan okumanın adından yola çıkarak çerezleri kontrol etmeye karar verdim.

![](2.webp){: width="1200" height="600" }

Geliştirici araçlarını açtıktan sonra uygulama sekmesini açtım ve çerezleri seçtim. Burada isAdmin adında bir parametre bulunuyor, içerdiği değer ise 0. Bildiğiniz üzere bilgisayar dilinde 0 olumsuz bir ifadedir. Kısaca 1 olarak değiştirdiğimde admin yetkisine sahip olacağım.

![](3.webp){: width="1200" height="600" }
