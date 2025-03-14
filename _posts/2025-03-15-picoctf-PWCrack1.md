---
title: 'picoCTF 2022 | PW Crack 1 WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "PW Crack 1" isimli meydan okumanın çözümünü göstereceğim. "level1.flag.txt.enc" ve "level1.py" adında iki dosya verilmiş ve bizden python dosyasını kullanarak bayrağı elde etmemizi istiyor.

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

flag_enc = open('level1.flag.txt.enc', 'rb').read()

def level_1_pw_check():
    user_pw = input("Please enter correct password for flag: ")
    if( user_pw == "691d"):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), user_pw)
        print(decryption)
        return
    print("That password is incorrect")

level_1_pw_check()
```

İlk satırda gördüğümüz açıklamada str_xor() fonksiyonunu görmezden gelmemiz gerektiği yazıyor. Öyle de yapıyor ve level_1_pw_check() fonksiyonunu incelemeye başlıyoruz. İlk iş olarak kullanıcıdan bir parola alıyor ve bunu user_pw değişkenine atıyor. Daha sonra ise girilen parola ile elindekini karşılaştırıyor. Eğer parola eşleşiyorsa parolayı str_xor() fonksiyonuna gönderiyor ve parolayı çözüp ekrana yazdırıyor.

```console
C:\Users\Appdone\Desktop>python level1.py
Please enter correct password for flag: 691d
Welcome back... your flag, user:
picoCTF{*******************}
```
