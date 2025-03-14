---
title: 'picoCTF 2022 | convertme.py WriteUp'
author: appdone
categories: [picoCTF 2022 Challenges]
tags: [binary, decimal]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan “convertme.py” isimli meydan okumanın çözümünü göstereceğim. Verilen python dosyasının içeriği aşağıdaki şekildedir.

```py
import random

def str_xor(secret, key):
    #extend key to secret length
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])

flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5f) + chr(0x05) + chr(0x08) + chr(0x2a) + chr(0x1c) + chr(0x5e) + chr(0x1e) + chr(0x1b) + chr(0x3b) + chr(0x17) + chr(0x51) + chr(0x5b) + chr(0x58) + chr(0x5c) + chr(0x3b) + chr(0x42) + chr(0x57) + chr(0x5c) + chr(0x0d) + chr(0x5f) + chr(0x06) + chr(0x46) + chr(0x5c) + chr(0x13)

num = random.choice(range(10,101))

print('If ' + str(num) + ' is in decimal base, what is it in binary base?')

ans = input('Answer: ')

try:
  ans_num = int(ans, base=2)
  
  if ans_num == num:
    flag = str_xor(flag_enc, 'enkidu')
    print('That is correct! Here\'s your flag: ' + flag)
  else:
    print(str(ans_num) + ' and ' + str(num) + ' are not equal.')
  
except ValueError:
  print('That isn\'t a binary number. Binary numbers contain only 1\'s and 0\'s')
```

Gördüğünüz üzere 10 ile 100 arasında rastgele bir sayı seçiyor ve bunu ekrana yazdırıyor. Burada istenen şey seçilen sayının ikilik gösterimidir.

```console
C:\Users\Appdone\Desktop>python convertme.py
If 14 is in decimal base, what is it in binary base?
Answer:
```

14 sayısının ikilik sayı sistemindeki halini girmemizi istiyor. Bunu yapabilmek için 14 sayısını bölünemeyene kadar 2'ye bölmemiz ve kalanları sondan başlayarak yan yana getirmemiz gerekiyor.

Örneğin: 18 sayısının ikilik sayı sistemindeki gösterimi aşağıdaki şekildedir.
- 18 / 2 = 9, kalan 0
- 9 / 2 = 4, kalan 1
- 4 / 2 = 2, kalan 0
- 2 / 2 = 1, kalan 0
- 1 / 2 = 0, kalan 1

Ters sıra ile yan yana koyduğumuzda sonuç 10010 olur. Aynı işlemi bize verilen sayı için yaptığımızda istenilen ikilik sayıyı elde etmiş oluyoruz.

- 14 / 2 = 7, kalan 0
- 7 / 2 = 3, kalan 1
- 3 / 2 = 1, kalan 1
- 1 / 2 =0, kalan 1

Sonuç olarak 14 sayısının ikilik sayı sistemindeki gösterimini 1110 olarak bulduk.

```console
C:\Users\Appdone\Desktop>python convertme.py
If 14 is in decimal base, what is it in binary base?
Answer: 1110
That is correct! Here's your flag: picoCTF{**************}
```
