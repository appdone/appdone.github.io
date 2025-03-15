---
title: 'picoCTF 2022 | Vigenere WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [cryptography]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Vigenere" isimli meydan okumayı çözeceğiz. Kodlanmış bir metin ve bir parola verilmiş. Bu parolayı kullanarak metni çözmemiz gerektiği belirtilmiş. Chepy aracından yararlanarak kolayca çözüyoruz.

```console
$ chepy "rgnoDVD{O0NU_WQ3_G1G3O3T3_A1AH3S_f85729e7}"
>>> vigenere_decode
ERROR: The function received no value for the required argument: key
Usage: chepy 'rgnoDVD{O0NU_WQ3_G1G3O3T3_A1AH3S_f85729e7}' - vigenere_decode <group> | --key=KEY
  available groups:      FIRE_METADATA

For detailed information on this command, run:
  chepy 'rgnoDVD{O0NU_WQ3_G1G3O3T3_A1AH3S_f85729e7}' - vigenere_decode --help
>>> vigenere_decode --help
INFO: Showing help with the command 'chepy '"'"'rgnoDVD{O0NU_WQ3_G1G3O3T3_A1AH3S_f85729e7}'"'"' - vigenere_decode -- --help'.

>>> vigenere_decode --key CYLAB
picoCTF{********************}
>>>
```
