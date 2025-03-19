---
title: 'TryHackMe | CyberHeroes WriteUp'
author: appdone
categories: [TryHackMe Challenges]
render_with_liquid: false
media_subpath: /images/tryhackme_cyberheroes/
image:
  path: logo.webp
---

Merhaba, bu yazımda TryHackMe platformunda yer alan “CyberHeroes” isimli meydan okumanın çözümünü göstereceğim. Meydan okumanın açıklamasında kendimi ispatlayabilmem için giriş yapmanın bir yolunu bulmam gerektiği yazıyor.

![](1.webp){: width="1200" height="600" }

Hızlı bir göz gezdirmenin ardından login.html sayfasının kaynak kodunda aşağıdaki javascript kodunu buldum.

```js
function authenticate() {
    a = document.getElementById('uname');
    b = document.getElementById('pass');
    const RevereString = str => [...str].reverse().join('');
    
    if (a.value == "h3ck3rBoi" & b.value == RevereString("54321@terceSrepuS")) {
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
            if (this.readyState == 4 && this.status == 200) {
                document.getElementById("flag").innerHTML = this.responseText;
                document.getElementById("todel").innerHTML = "";
                document.getElementById("rm").remove();
            }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_" + a.value + "_" + b.value + ".txt", true);
        xhttp.send();
    } else {
        alert("Incorrect Password, try again.. you got this hacker!");
    }
}
```

Burada kullanıcı adı ve parolanın doğrulandığını görüyoruz. Doğrulama sonunda ise bayrağı bir dosyadan alıp bize veriyor. Kullanıcı adını doğrudan kopyalayıp forma ekliyorum. Parolayı ise ters çevirip form’a ekliyorum ve giriş yaptıktan sonra bayrağı bize veriyor.

```
Kullanıcı adı: h3ck3rBoi
Parola: SuperSecret@12345
```

![](2.webp){: width="1200" height="600" }

Bu meydan okumayı bir kaç ay önce abonelere özel kısma eklemişlerdi. Nedendir bilinmez tekrar herkese açık hale getirmişler. Herhalde aboneler için fazla sıkıcı olduğunu düşündüler.
