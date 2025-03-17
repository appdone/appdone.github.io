---
title: 'picoCTF 2022 | Redaction gone wrong WriteUp'
author: appdone
categories: [picoCTF 2022 Challenges]
render_with_liquid: false
media_subpath: /images/picoctf_redaction_gone_wrong/
image:
  path: 1.webp
---

Bu yazıda, picoCTF platformunda yer alan "Redaction gone wrong" isimli meydan okumayı çözeceğiz. Bu sefer elimizde bir PDF dosyası var.

![pdf](1.webp){: width="1200" height="600" }

Bu şekilde önemli verilerin bulunduğu kısımlar gizlenmiş. Bende öyle gözükmüyor biliyorum ama önceden PDF dosyalarının içerisindeki verileri renkler ile vurgulayabiliyordum. Eğer sadece vurgu rengi siyah yapılarak gizlenmiş ise CTRL + A kombinasyonunu kullanarak doğrudan bayrağı kopyalayabiliriz.

![pdf](2.webp){: width="1200" height="600" }
