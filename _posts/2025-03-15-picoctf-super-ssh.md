---
title: 'picoCTF 2022 | Super SSH WriteUp'
author: 'appdone'
categories: [picoCTF 2024 Challenges]
tags: [ssh]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Super SSH" isimli meydan okumayı çözeceğiz. Kolay seviye meydan okumalardan biri kendisi. Öyleki bayrağı alabilmek için SSH servisine bağlanmamız yetiyor.

```console
C:\Users\Appdone>ssh ctf-player@titan.picoctf.net -p 56492
The authenticity of host '[titan.picoctf.net]:56492 ([3.139.174.234]:56492)' can't be established.
ED25519 key fingerprint is SHA256:4S9EbTSSRZm32I+cdM5TyzthpQryv5kudRP9PIKT7XQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[titan.picoctf.net]:56492' (ED25519) to the list of known hosts.
ctf-player@titan.picoctf.net's password:
Welcome ctf-player, here's your flag: `picoCTF{*********************}`
Connection to titan.picoctf.net closed.
```
