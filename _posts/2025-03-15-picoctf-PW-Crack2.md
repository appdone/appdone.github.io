---
title: 'picoCTF 2022 | PW Crack 2 WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "PW Crack 2" isimli meydan okumanın çözümünü göstereceğim. "level2.flag.txt.enc" ve "level2.py" adında iki dosya verilmiş ve kodu analiz edip, bayrağa erişmemiz isteniyor.

```py
### THIS FUNCTION WILL NOT HELP YOU FIND THE FLAG --LT ########################
def str_xor(secret, key):
    #extend key to secret length
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])
###############################################################################

flag_enc = open('level2.flag.txt.enc', 'rb').read()

def level_2_pw_check():
    user_pw = input("Please enter correct password for flag: ")
    if( user_pw == chr(0x34) + chr(0x65) + chr(0x63) + chr(0x39) ):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), user_pw)
        print(decryption)
        return
    print("That password is incorrect")

level_2_pw_check()
```

Açıklama satırları içerisinde str_xor() fonksiyonu ile ilgilenmemiz gerektiği yazıyor. level_2_pw_check() fonksiyonuna baktığımızda ise bir parola istediğini ve girilen parolayı hex karakterlerinin karşılığının birleşmiş haliyle karşılaştırdığını görüyoruz.

Python konsolunu açtıktan sonra hexdecimal veriyi alıp yapıştırdığımızda parolayı görüyoruz. Aslında sorguyu kaldırsak bile aynı şekilde bayrağı bulacağız.

```console
C:\Users\Appdone\Desktop>python
Python 3.11.4 (tags/v3.11.4:d2340ef, Jun  7 2023, 05:45:37) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> chr(0x34) + chr(0x65) + chr(0x63) + chr(0x39)
'4ec9'
>>> exit()

C:\Users\Appdone\Desktop>python level2.py
Please enter correct password for flag: 4ec9
Welcome back... your flag, user:
picoCTF{**********************}
```
