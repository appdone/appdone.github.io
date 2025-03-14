---
title: 'picoCTF 2022 | HashingJobApp WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [md5]
render_with_liquid: false
---

Merhaba, bu yazımda picoCTF platformunda yer alan "HashingJobApp" isimli meydan okumanın çözümünü göstereceğim. Meydan okumayı başlattıktan sonra verilen host ve port'a ncat aracılığıyla bağlanıyoruz ve bizden bir kelimenin md5 ile hash'lenmiş halini istediğini görüyoruz.

CyberChef sitesinin yardımıyla verilen kelimeyi hash'ledikten sonra giriyoruz. Daha sonra bir kaç kelimenin daha hash'lenmiş halini istedi. Her defasında aynı işlemi yaptıktan sonra bayrağa ulaşıyoruz.

```console
C:\Users\Appdone\Desktop>ncat saturn.picoctf.net 54882
Please md5 hash the text between quotes, excluding the quotes: 'jelly beans'
Answer:
cb7d86ece76c57eac5ed18420ca67ea0
cb7d86ece76c57eac5ed18420ca67ea0
Correct.
Please md5 hash the text between quotes, excluding the quotes: 'gym teachers'
Answer:
6a8404f911c6543cada93a75dd30a57d
6a8404f911c6543cada93a75dd30a57d
Correct.
Please md5 hash the text between quotes, excluding the quotes: 'hockey'
Answer:
df0349ce110b69f03b4def8012ae4970
df0349ce110b69f03b4def8012ae4970
Correct.
picoCTF{********************}
```
