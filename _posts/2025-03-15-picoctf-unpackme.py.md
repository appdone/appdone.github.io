---
title: 'picoCTF 2022 | convertme.py WriteUp'
author: appdone
categories: [picoCTF 2022]
tags: [binary, decimal]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "unpackme.py" isimli meydan okumanın çözümünü göstereceğim. Verilen python dosyasının içeriği aşağıdaki şekildedir.

```py
import base64
from cryptography.fernet import Fernet

payload = b'gAAAAABkzWGSzE6VQNTzvRXOXekQeW4CY6NiRkzeImo9LuYBHAYw_hagTJLJL0c-kmNsjY33IUbU2IWlqxA3Fpp9S7RxNkiwMDZgLmRlI9-lGAEW-_i72RSDvylNR3QkpJW2JxubjLUC5VwoVgH62wxDuYu1rRD5KadwTADdABqsx2MkY6fKNTMCYY09Se6yjtRBftfTJUL-LKz2bwgXNd6O-WpbfXEMvCv3gNQ7sW4pgUnb-gDVZvrLNrug_1YFaIe3yKr0Awo0HIN3XMdZYpSE1c9P4G0sMQ=='

key_str = 'correctstaplecorrectstaplecorrec'
key_base64 = base64.b64encode(key_str.encode())
f = Fernet(key_base64)
plain = f.decrypt(payload)
exec(plain.decode())
```

Son satırdaki "exec" yazısının adını print olarak değiştirdim. Böylece çözülen kodu çalıştırmak yerine ekrana yazdırıyor.

```console
C:\Users\Appdone\Desktop>python unpackme.flag.py

pw = input('What\'s the password? ')

if pw == 'batteryhorse':
  print('picoCTF{175_chr157m45_cd82f94c}')
else:
  print('That password is incorrect.')
```
