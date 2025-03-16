---
title: 'TryHackMe | Capture! WriteUp'
author: appdone
categories: [TryHackMe]
render_with_liquid: false
media_subpath: /images/tryhackme_capture/
image:
  path: logo.webp
---

Bu yazımda, TryHackMe platformunda yer alan "Capture!" isimli meydan okumayı çözeceğiz. Bizden istenen şey verilen kullanıcı isimlerini ve parolalarını kullanarak korunan bir giriş sayfasına deneme yanılma saldırısı yapmaktır.

![login page](1.webp){: width="900" height="500" }

Yanlış kullanıcı adı girildiğinde kullanıcının bulunmadığını söylüyor. Bununla birlikte bir dizi deneme yaptıktan sonra ise aşağıdaki doğrulama yöntemi ile karşılaşıyoruz.

![login page](2.webp){: width="900" height="500" }

Bu güvenlik önlemini geçebilmek için deneme yanılma saldırısı yaparken doğrulama sorusunu çözecek bir uygulama yapmamız gerekiyor.

Python programlama dilini kullanarak hızlıca bir deneme yanılma aracı oluşturdum ve çalıştırdıktan sonra kullanıcı bilgilerini elde ettim. Kodlara [buradan](https://github.com/appdone/CTF-Tools/blob/main/TryHackMe/capture.py) ulaşabilirsiniz.

https://github.com/appdone/CTF-Tools/blob/main/TryHackMe/capture.py

```console
$ python3 capture.py 
username: *** - password: ***
```

Programın kullanıcı adını ve parolasını bulması uzun sürebilir, biraz sabretmemiz gerekiyor. Bulduktan sonra da bayrağı alabilmek için tek yapmamız gereken şey siteye giriş yapmaktır.
