---
title: 'TryHackMe | TakeOver WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [subdomain enumeration]
render_with_liquid: false
media_subpath: /images/tryhackme_takeover/
image:
  path: logo.webp
---

## Özet

DNSMap aracından yararlanarak hedef domaine ait olan subdomainleri tespit edecek ve bu subdomainlerden birinin sertifikasından yararlanarak bayrağın bulunduğu adresi öğreneceğiz.

## Keşif aşaması

### 80/443 numaralı portun incelenmesi

![](1.webp){: width="1200" height="600" }

Subdomainler ile ilgilenmemiz gerektiği belirtilmiş. Doğrudan dnsmap aracını kullanarak hedef uygulamanın subdomainlerini tespit edelim.

```console
$ dnsmap futurevera.thm
dnsmap 0.36 - DNS Network Mapper

[+] searching (sub)domains for futurevera.thm using built-in wordlist
[+] using maximum random delay of 10 millisecond(s) between requests

blog.futurevera.thm
IP address #1: 10.10.191.95
[+] warning: internal IP address disclosed

portal.futurevera.thm
IP address #1: 10.10.191.95
[+] warning: internal IP address disclosed

support.futurevera.thm
IP address #1: 10.10.191.95
[+] warning: internal IP address disclosed

[+] 3 (sub)domains and 3 IP address(es) found
[+] 3 internal IP address(es) disclosed
[+] completion time: 225 second(s)
```

Tarama sonucunda üç adet subdomaini tespit ettiğini görüyoruz. Her birini /etc/hosts dosyasına girdikten sonra subdomainleri tek tek incelemeye başladım. `support.futurevera.thm` adresinin sertifikasını görüntülediğimizde onun da altında bir subdomain olduğunu görüyoruz.

![](2.webp){: width="1200" height="600" }

Bu subdomainide /etc/hosts dosyasına girdikten sonra ziyaret ettiğimizde bizi bayrağın url yapısında bulunduğu boş bir adrese yönlendiriyor.

```console
$ curl secrethelpdesk934752.support.futurevera.thm -i
HTTP/1.1 302 Found
Date: Thu, 27 Mar 2025 20:52:15 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: http://flag{beea0d6***ae81b2f}.s3-website-us-west-3.amazonaws.com/
Content-Length: 0
Content-Type: text/html; charset=UTF-8
``` 
