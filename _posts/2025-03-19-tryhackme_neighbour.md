---
title: 'TryHackMe | Neighbour WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [idor]
render_with_liquid: false
media_subpath: /images/tryhackme_neighbour/
image:
  path: logo.webp
---

Bu yazımda TryHackMe platformunda yer alan “Neighbour” isimli meydan okumanın çözümünü göstereceğim. Kendisi IDOR zafiyetinin pratiğini yapabileceğimiz kolay bir meydan okuma.

![](1.webp){: width="1200" height="600" }

Giriş sayfasında misafir kullanıcı olarak giriş yapabileceğimizden bahseden bir yazı ile karşılaşıyoruz. Sayfanın kaynak kodunu görüntülediğimizde bununla ilgili bir şeyler görebileceğimizden bahsediyor.

![](2.webp){: width="1200" height="600" }

Kaynak kodunu görüntülediğimizde ise kayıt işlemleri tamamlanana kadar “guest:guest” kullanıcı bilgilerini kullanabileceğimizi söylediğini görüyoruz. Bununla beraber “admin” kullanıcısının varlığından da haberdar oluyoruz. Elimizdeki bilgileri kullanarak giriş yapalım.

![](3.webp){: width="1200" height="500" }

Misafir olarak giriş yaptıktan sonra bizi yukarıdaki sayfa karşıladı. Url yapısına baktığımızda user parametresi ile “guest” kullanıcısının profilini görüntülediğini görüyoruz. Kullanıcı adını “admin” olarak değiştirdiğimizde ise admin kullanıcısının sayfasını gösteriyor ve böylece bayrağı kolayca buluyoruz.

![](4.webp){: width="1200" height="500" }
