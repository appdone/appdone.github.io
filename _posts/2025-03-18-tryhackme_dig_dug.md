---
title: 'TryHackMe | Dig Dug WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [dns]
render_with_liquid: false
---

Merhaba arkadaşlar, bu yazımda TryHackMe platformunda bulunan “Dig Dug” isimli meydan okumanın çözümünü göstereceğim. Kendisi tek hamlede çözülebilecek bir meydan okumadır.

Meydan okumanın açıklamasında hedef makinenin bir DNS sunucu olduğu ve sadece givemetheflag.com adresi için gelen sorguları yanıtladığı yazıyor. Bu durumda aşağıdaki komutu kullanarak bayrağın bulunabileceği kayıtları görüntüleyebilmemiz gerekiyor.

```console
$ dig @10.10.35.7 givemetheflag.com

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> @10.10.35.7 givemetheflag.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47579
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;givemetheflag.com.             IN      A

;; ANSWER SECTION:
givemetheflag.com.      0       IN      TXT     "flag{***}"

;; Query time: 73 msec
;; SERVER: 10.10.35.7#53(10.10.35.7) (UDP)
;; WHEN: Mon Mar 17 18:17:05 EDT 2025
;; MSG SIZE  rcvd: 86

```
