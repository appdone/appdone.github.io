---
title: 'picoCTF 2022 | patchme.py WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "patchme.py" isimli meydan okumayı çözeceğiz. Elimizde "patchme.flag.py" ve "flag.txt.enc" olmak üzere iki dosya var. Bayrağın bulunduğu dosyada anlamsız yazılar bulunuyor, python dosyasının içeriği ise aşağıdaki şekilde.

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

flag_enc = open('flag.txt.enc', 'rb').read()

def level_1_pw_check():
    user_pw = input("Please enter correct password for flag: ")
    if( user_pw == "ak98" + \
                   "-=90" + \
                   "adfjhgj321" + \
                   "sleuth9000"):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), "utilitarian")
        print(decryption)
        return
    print("That password is incorrect")
    
level_1_pw_check()
```

Python dosyasını çalıştırdığımızda ilk "olarak level_1_pw_check()" isimli fonksiyon çalışacak. Bu fonksiyonda önce parola sorulduğunu, parola doğru ise bayrağın çözülüp ekrana yazdırıldığını görüyoruz. Parolayıda vermiş zaten, sadece birleştirmemiz gerekiyor.

```console
C:\Users\Appdone\Desktop>python patchme.flag.py
Please enter correct password for flag: ak98-=90adfjhgj321sleuth9000
Welcome back... your flag, user:
picoCTF{**********************}
```
