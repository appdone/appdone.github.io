---
title: 'picoCTF 2022 | Glitch Cat WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "Glitch Cat" isimli meydan okumanın çözümünü göstereceğim. Meydan okumayı başlattıktan sonra verilen host ve port'a nc aracılığıyla bağlanıyoruz. Bağlandıktan sonra aşağıdaki şekilde bayrağı buluyoruz.

```
'picoCTF{gl17ch_m3_n07_' + chr(0x39) + chr(0x63) + chr(0x34) + chr(0x32) + chr(0x61) + chr(0x34) + chr(0x35) + chr(0x64) + '}'
```

Python konsolunu açtıktan sonra elimizdeki kodu yapıştırıyor ve böylece bayrağı elde etmiş oluyoruz.

```console
C:\Users\Appdone\Desktop>python
Python 3.11.4 (tags/v3.11.4:d2340ef, Jun  7 2023, 05:45:37) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> 'picoCTF{gl17ch_m3_n07_' + chr(0x39) + chr(0x63) + chr(0x34) + chr(0x32) + chr(0x61) + chr(0x34) + chr(0x35) + chr(0x64) + '}'
'picoCTF{gl17ch_m3_n07_********}'
>>> exit()
```
