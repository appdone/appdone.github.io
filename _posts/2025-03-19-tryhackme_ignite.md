---
title: 'TryHackMe | Ignite WriteUp'
author: appdone
categories: [TryHackMe Challenges]
render_with_liquid: false
media_subpath: /images/tryhackme_ignite/
image:
  path: logo.webp
---

Bu yazımda, TryHackMe platformunda yer alan “Ignite” isimli meydan okumanın çözümünü göstereceğim. Web sitesini açtığımızda Fuel CMS isimli içerik yönetim sisteminin kurulum sayfası ile karşılaşıyoruz. Henüz bir kurulum işlemi gerçekleştirilmediğinden yönetim paneline girmek için varsayılan kullanıcı bilgilerini kullanabiliriz.

![](1.webp){: width="900" height="500" }

Yönetim paneline giriş yaptıktan sonra dosya yükleyebileceğim alanları zorlamaya başladım. Ancak bir türlü dosya yükleyemedim. Daha sonra Fuel CMS’in sürümünü google da arattığımda [bu](https://github.com/noraj/fuelcms-rce/blob/master/exploit.rb) exploit’i buldum.

Doğrudan kullanmak yerine kullandığı url yapısını düzenledim ve 1234 numaralı port üzerinden bir shell bağlantısı aldım.

![](2.webp){: width="900" height="500" }

Yönetim panelinde bulunan users kategorisini kontrol ettiğimden sadece admin kullanıcısının kullanıldığını biliyorum ama yinede konfigürasyon dosyasındaki parolayı denemem gerekiyor, zira sistemdeki bir kullanıcının parolası olabilir.

```
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => '***',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        ...
);
```

Bulduğum parolayı sistemde bulunan tek kullanıcıya yani root kullanıcısına geçiş yapmak için kullandım.

```console
www-data@ubuntu:/var/www/html/fuel/application/config$ su root
Password: 
root@ubuntu:/var/www/html/fuel/application/config# cd /root
root@ubuntu:~# ls
root.txt
```
