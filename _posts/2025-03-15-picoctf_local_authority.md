---
title: 'picoCTF 2022 | Local Authority WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
media_subpath: /images/picoctf_local_authority/
image:
  path: 1.webp
---

Bu yazıda, picoCTF platformunda yer alan "Local Authority" isimli meydan okumayı çözeceğiz.

![login page](1.webp){: width="1200" height="400" }

Web sitesinde bir giriş formu bulunuyor. Rastgele bilgiler girdikten sonra bizi login.php sayfasına yönlendiriyor. Orada giriş yapılamadı hatası ile karşılaşıyoruz. login.php sayfasının kaynak kodunu görüntülediğimizde ise bir takım javascript kodları ile karşılaşıyoruz.

![source code](2.webp){: width="1200" height="600" }

Sayfa içerisindeki javascript kodlarında bir sorun yok. Dışarıdan dahil edilen javascript kodlarında ise kullanıcı bilgilerini buluyoruz.

```js
function checkPassword(username, password)
{
  if( username === 'admin' && password === '*******' )
  {
    return true;
  }
  else
  {
    return false;
  }
}
```

Son olarak elde ettiğimiz bilgileri kullanmak için giriş sayfasına yöneliyoruz ve giriş yaptıktan sonra bayrağı görüyoruz.
