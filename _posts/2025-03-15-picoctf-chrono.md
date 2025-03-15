---
title: 'picoCTF 2022 | convertme.py WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [binary, decimal]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "Chrono" isimli meydan okumayı çözeceğiz. Meydan okumanın açıklamasında linux sistemlerde belli aralıklarla çalışan görevleri nasıl oluşturabileceğimizi soruyor. Sorunun cevabına istinaden /etc/crontab dosyasını görüntülediğimizde bayrağa ulaşıyoruz.

```console
picoplayer@challenge:~$ cat /etc/crontab
# picoCTF{*********************************}

```
