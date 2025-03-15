---
title: 'picoCTF 2022 | Chrono WriteUp'
author: 'appdone'
categories: [picoCTF 2023 Challenges]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "Chrono" isimli meydan okumayı çözeceğiz. Meydan okumanın açıklamasında linux sistemlerde belli aralıklarla çalışan görevleri nasıl oluşturabileceğimizi soruyor. Sorunun cevabına istinaden /etc/crontab dosyasını görüntülediğimizde bayrağa ulaşıyoruz.

```console
picoplayer@challenge:~$ cat /etc/crontab
# picoCTF{*********************************}

```
