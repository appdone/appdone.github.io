---
title: 'picoCTF 2023 | useless WriteUp'
author: 'appdone'
categories: [picoCTF 2023 Challenges]
render_with_liquid: false
---

Bu yazımda, picoCTF platformunda yer alan "useless" isimli meydan okumayı çözeceğiz. Meydan okumanın açıklamasında kullanıcı dizininin altında bir dosya olduğundan bahsediyor ve makineyi başlattıktan sonra SSH servisine bağlanabilmemiz için kullanıcı bilgileri veriyor.

```console
picoplayer@challenge:~$ ls -la
total 16
drwxr-xr-x 1 picoplayer picoplayer   20 Mar 15 09:48 .
drwxr-xr-x 1 root       root         24 Aug  4  2023 ..
-rw-r--r-- 1 picoplayer picoplayer  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 picoplayer picoplayer 3771 Feb 25  2020 .bashrc
drwx------ 2 picoplayer picoplayer   34 Mar 15 09:48 .cache
-rw-r--r-- 1 picoplayer picoplayer  807 Feb 25  2020 .profile
-rwxr-xr-x 1 root       root        517 Mar 16  2023 useless
picoplayer@challenge:~$ cat useless
#!/bin/bash
# Basic mathematical operations via command-line arguments

if [ $# != 3 ]
then
  echo "Read the code first"
else
        if [[ "$1" == "add" ]]
        then
          sum=$(( $2 + $3 ))
          echo "The Sum is: $sum"

        elif [[ "$1" == "sub" ]]
        then
          sub=$(( $2 - $3 ))
          echo "The Substract is: $sub"

        elif [[ "$1" == "div" ]]
        then
          div=$(( $2 / $3 ))
          echo "The quotient is: $div"

        elif [[ "$1" == "mul" ]]
        then
          mul=$(( $2 * $3 ))
          echo "The product is: $mul"

        else
          echo "Read the manual"

        fi
fi
```

Alt satırda bulunan "Read the manual" mesajı dışında ilginç bir şey göremedim. Yanlış üç parametre girdiğimizde bu mesajı veriyor. Detayları da görüntüleyebileceğimiz başka sayfa olmadığından "man useless" komutunu kullandım ve en alt satırda bayrağı gördüm.

```console
picoplayer@challenge:~$ man useless

useless
     useless, -- This is a simple calculator script

SYNOPSIS
     useless, [add sub mul div] number1 number2

DESCRIPTION
     Use the useless, macro to make simple calulations like addition,subtraction, multiplication and division.

Examples
     ./useless add 1 2
       This will add 1 and 2 and return 3

     ./useless mul 2 3
       This will return 6 as a product of 2 and 3

     ./useless div 6 3
       This will return 2 as a quotient of 6 and 3

     ./useless sub 6 5
       This will return 1 as a remainder of substraction of 5 from 6

Authors
     This script was designed and developed by Cylab Africa

     picoCTF{****************************}

picoplayer@challenge:~$
```
