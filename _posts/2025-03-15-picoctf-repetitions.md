---
title: 'picoCTF 2022 | convertme.py WriteUp'
author: Appdone
categories: [picoCTF 2023]
tags: [base64]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "Repetitions" isimli meydan okumanın çözümünü göstereceğim. Bize bir dosya verilmiş ve bu dosyanın içerisindeki bayrağı almamızı istiyor. enc_flag isimli dosyanın içeriği aşağıdaki gibidir.

```
VmpGU1EyRXlUWGxTYmxKVVYwZFNWbGxyV21GV1JteDBUbFpPYWxKdFVsaFpWVlUxWVZaS1ZWWnVh
RmRXZWtab1dWWmtSMk5yTlZWWApiVVpUVm10d1VWZFdVa2RpYlZaWFZtNVdVZ3BpU0VKeldWUkNk
MlZXVlhoWGJYQk9VbFJXU0ZkcVRuTldaM0JZVWpGS2VWWkdaSGRXCk1sWnpWV3hhVm1KRk5XOVVW
VkpEVGxaYVdFMVhSbHBWV0VKVVZGWmFWMDVHV2tkYVNHUlZDazFyY0ZkVWJGWlhZVlpLU0dWRlZs
aGkKYlRrelZERldUMkpzUWxWTlJYTkxDZz09Cg==
```

Base64 ile kodlanmış ve belirli aralıklarla yeni satıra geçmiş bir veriye benziyor. Satırları birleştirdikten ve çözdükten sonra tekrar bir base64 kodlaması ile karşılaşıyoruz. Bunu 6 kez tekrar ettikten sonra bayrağa ulaşıyoruz. Aşağıya bu işlemi otomatik yapan bir python kodu bırakıyorum.

```py
from base64 import b64decode

with open("enc_flag", "r") as file:
    enc_flag = file.read()

while "picoCTF" not in str(enc_flag):
    enc_flag = str(enc_flag).replace('\n', '')
    enc_flag = b64decode(enc_flag).decode()

print(enc_flag)
```
