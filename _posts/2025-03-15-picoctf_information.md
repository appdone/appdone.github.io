---
title: 'picoCTF 2021 | Information WriteUp'
author: 'appdone'
categories: [picoCTF 2021 Challenges]
render_with_liquid: false
---

Merhaba, bu yazıda picoCTF platformunda yer alan "Information" isimli meydan okumayı çözeceğiz. Öncelikle verilen resim dosyasının kaynak kodunu görüntüledim ve içerisinde bayrağı aradım ama bulamadım. Daha sonra exiftool aracılığıyla dosyanın meta verilerini görüntülediğimde "license" parametresinde bir base64 kodlaması olduğunu fark ettim.

```console
$ exiftool cat.jpg 
ExifTool Version Number         : 12.57
File Name                       : cat.jpg
Directory                       : .
File Size                       : 878 kB
File Modification Date/Time     : 2025:03:15 08:39:07-04:00
File Access Date/Time           : 2025:03:15 08:39:10-04:00
File Inode Change Date/Time     : 2025:03:15 08:39:07-04:00
File Permissions                : -rw-rw-rw-
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.02
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Current IPTC Digest             : 7a78f3d9cfb1ce42ab5a3aa30573d617
Copyright Notice                : PicoCTF
Application Record Version      : 4
XMP Toolkit                     : Image::ExifTool 10.80
License                         : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
Rights                          : PicoCTF
Image Width                     : 2560
Image Height                    : 1598
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2560x1598
Megapixels                      : 4.1
```

Bulduğumuz base64 kodlamasını Chepy aracının yardımıyla çözdüğümüzde bayrağı elde ediyoruz.

```console
$ chepy "cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9"
>>> from_base64
picoCTF{***}
```
