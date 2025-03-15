---
title: 'picoCTF 2024 | Bookmarklet WriteUp'
author: 'appdone'
categories: [picoCTF 2024 Challenges]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Bookmarklet" isimli meydan okumayı çözeceğiz. Web uygulamasında kodlanmış bir veriyi çözen bir javascript kodu bulunuyor.

```js
javascript:(function() {
    var encryptedFlag = "àÒÆÞ¦È¬ëÙ£ÖÓÚåÛÑ¢ÕÓ¡ÒÅ¤í";
    var key = "picoctf";
    var decryptedFlag = "";
    for (var i = 0; i < encryptedFlag.length; i++) {
         decryptedFlag += String.fromCharCode((encryptedFlag.charCodeAt(i) - key.charCodeAt(i % key.length) + 256) % 256);
    }
    alert(decryptedFlag);
})(); 
```

Bu javascript kodunu kopyalayıp geliştirici araçlarındaki konsolda çalıştırdığımızda bayrağa ulaşıyoruz.
