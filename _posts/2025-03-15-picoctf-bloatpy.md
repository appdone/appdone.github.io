---
title: 'picoCTF 2022 | bloat.py WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [obfuscate, deobfuscate]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "bloat.py" isimli meydan okumayı çözeceğiz. "flag.txt.enc" ve "bloat.flag.py" olmak üzere iki dosya verilmiş. Python dosyasının içeriğini aşağıda görüyorsunuz.

```py
import sys
a = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \
            "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~ "
def arg133(arg432):
  if arg432 == a[71]+a[64]+a[79]+a[79]+a[88]+a[66]+a[71]+a[64]+a[77]+a[66]+a[68]:
    return True
  else:
    print(a[51]+a[71]+a[64]+a[83]+a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+\
a[81]+a[67]+a[94]+a[72]+a[82]+a[94]+a[72]+a[77]+a[66]+a[78]+a[81]+\
a[81]+a[68]+a[66]+a[83])
    sys.exit(0)
    return False
def arg111(arg444):
  return arg122(arg444.decode(), a[81]+a[64]+a[79]+a[82]+a[66]+a[64]+a[75]+\
a[75]+a[72]+a[78]+a[77])
def arg232():
  return input(a[47]+a[75]+a[68]+a[64]+a[82]+a[68]+a[94]+a[68]+a[77]+a[83]+\
a[68]+a[81]+a[94]+a[66]+a[78]+a[81]+a[81]+a[68]+a[66]+a[83]+\
a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+a[81]+a[67]+a[94]+\
a[69]+a[78]+a[81]+a[94]+a[69]+a[75]+a[64]+a[70]+a[25]+a[94])
def arg132():
  return open('flag.txt.enc', 'rb').read()
def arg112():
  print(a[54]+a[68]+a[75]+a[66]+a[78]+a[76]+a[68]+a[94]+a[65]+a[64]+a[66]+\
a[74]+a[13]+a[13]+a[13]+a[94]+a[88]+a[78]+a[84]+a[81]+a[94]+a[69]+\
a[75]+a[64]+a[70]+a[11]+a[94]+a[84]+a[82]+a[68]+a[81]+a[25])
def arg122(arg432, arg423):
    arg433 = arg423
    i = 0
    while len(arg433) < len(arg432):
        arg433 = arg433 + arg423[i]
        i = (i + 1) % len(arg423)        
    return "".join([chr(ord(arg422) ^ ord(arg442)) for (arg422,arg442) in zip(arg432,arg433)])
arg444 = arg132()
arg432 = arg232()
arg133(arg432)
arg112()
arg423 = arg111(arg444)
print(arg423)
sys.exit(0)
```

Gördüğünüz üzere kodun okunması zorlaşsın diye karmaşıklaştırmışlar yani obfuscate etmişler. Öncelikle bir karakter kümesi tanımlandığını görüyoruz.

```py
a = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \
            "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~ "
```

Bu karakterleri kullanarak verileri gizlemişler. Kodun ana kısmından incelemeye devam edelim.

```py
arg444 = arg132()
arg432 = arg232()
arg133(arg432)
arg112()
arg423 = arg111(arg444)
print(arg423)
sys.exit(0)
```

Öncelikle arg132() fonksiyonunu çalıştırıyor. Bu fonksiyonda flag.txt.enc dosyasını okuyor ve içeriğini "arg444" değişkenine atıyor. Daha sonra arg232() fonksiyonunu çalıştırarak kullanıcıdan bir veri aldığını görüyoruz. Aldığı veriyi arg133() fonksiyonuna gönderiyor ve veriyi kontrol ettiriyor. Girilen veri yanlışsa ekrana bir şeyler yazdırıp programı kapatıyor. Değilse okuduğu dosyadaki verileri çözüp ekrana yazdırıyor.

Burada önemli olan tek veri karşılaştırma işlemini gören veridir. Karşılaştırmanın başarılı olması için kendisini çözmemiz gerekiyor. Python konsolunu açtıktan sonra karakterler değişkenini ekleyelim. Daha sonra ise veriyi yapıştıralım.

```console
C:\Users\Appdone\Desktop>python
Python 3.11.4 (tags/v3.11.4:d2340ef, Jun  7 2023, 05:45:37) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> a = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \
...             "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~ "
>>> a[71]+a[64]+a[79]+a[79]+a[88]+a[66]+a[71]+a[64]+a[77]+a[66]+a[68]
'happychance'
>>> exit()
```

Parolayı aldıktan sonra programı çalıştırıyor ve bayrağı alıyoruz.

```console
C:\Users\Appdone\Desktop>python bloat.flag.py
Please enter correct password for flag: happychance
Welcome back... your flag, user:
picoCTF{**********************}
```
