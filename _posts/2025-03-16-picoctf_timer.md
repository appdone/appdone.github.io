---
title: 'picoCTF 2023 | Timer WriteUp'
author: appdone
categories: [picoCTF 2023 Challenges]
tags: [apk]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "Timer" isimli meydan okumayı çözeceğiz. Öncelikle verilen APK dosyasını dışarıya çıkaralım. Sonuçta APK dosyaları ZIP formatında oluşturulan dosyalardır.

```console
$ unzip timer.apk -d timer
Archive:  timer.apk                                                                                                                                                                           
  inflating: timer/classes3.dex                                                                                                                                                               
  inflating: timer/classes2.dex
... 
```

Elde ettiğimiz .dex dosyalarını, "dex2jar" aracını kullanarak JAR formatına çevirebilir daha sonra "jd-gui" gibi araçlarla JAR dosyalarının kaynak kodlarını görüntüleyebiliriz. Ancak bu çok uğraştırıcı olur. Bunun yerine dosyaların kaynağını doğrudan görüntüleyeceğim. Eğer bayrak dosyaların içerisinde bulunuyorsa uğraşmamıza gerek kalmaz ki öylede oluyor.

```console
$ cat * | strings | grep "picoCTF"
*picoCTF{***}
cat: kotlin: Is a directory
cat: META-INF: Is a directory
cat: res: Is a directory
```
