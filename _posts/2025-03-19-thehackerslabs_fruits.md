---
title: 'TheHackersLabs | Fruits WriteUp'
author: appdone
categories: [TheHackersLabs Challenges]
tags: [lfi, sudo]
render_with_liquid: false
media_subpath: /images/thehackerslabs_fruits/
image:
  path: 1.webp
---

Bugün yeni keşfettiğim “The Hackers Labs” platformundan kolay seviye bir CTF çözeceğim. Bu platformun Vulnhub’dan tek farkı, bayrak barındırıyor olmasıdır. user.txt ve root.txt olmak üzere iki bayrak var. Bunları aldıktan sonra platforma girerek puan toplayabiliyoruz.

Makineyi indirdikten sonra virtualbox ile açtım ve nmap ile bir port taraması gerçekleştirdim. Varsayılan 22 ve 80 portu açık olduğundan doğrudan HTTP servisine yöneleceğim.

![](1.webp){: width="1200" height="600" }

Herhangi bir veri girildiğinde var olmayan bir sayfaya yönlendiriyor. Bu durumda istediği veri türünün farklı olduğunu düşünebiliriz, ancak bu doğru değil. Hatayı var olmayan bir sayfaya yönlendirdiği için alıyoruz.

![](2.webp){: width="1200" height="400" }

Bir dizin taraması yaptım ama geçerli bir dizin bulamadı. Bu yüzden html ve php uzantılarını kullanarak tekrar bir tarama yapacağım.

![](3.webp){: width="1200" height="600" }

Tarama sırasında fruits.php adında bir dosya olduğunu gösterdi. İçerisinde herhangi bir veri olmadığı için bir kaç parametre denedim. Sonucunda “file” parametresi ile verilen dosyayı ekrana yazdırdığını öğrendim. Bu işlem sadece /etc/passwd dosyası için geçerli gibi görünüyor. user.txt veya id_rsa dosyasını çekemiyorum.

![](4.webp){: width="1200" height="600" }

Diğer dosyaları kabul etmediği için “bananaman” kullanıcı adını kullanarak, hydra ile deneme-yanılma saldırısı yapmaya karar verdim.

![](5.webp){: width="1200" height="600" }

Elde ettiğim parolayı kullanarak SSH servisine bağlandım ve user.txt dosyasından ilk bayrağı aldım. Bu arada, /etc/passwd dosyası dışındaki dosyaların ekrana yazdırılmamasının sebebini aşağıdaki kodda görebilirsiniz.

```php
<?php
// Obtener el nombre del archivo solicitado desde el parámetro 'file'
$file = $_GET['file'];

// Verificar si el archivo solicitado es "/etc/passwd"
if ($file === '/etc/passwd') {
    // Mostrar el contenido del archivo /etc/passwd
    echo "<pre>";
    echo file_get_contents('/etc/passwd');
    echo "</pre>";
} 
// No hacer nada si el archivo solicitado no es "/etc/passwd"
?>
```

Her neyse şimdi root.txt dosyasına erişebilmemiz için yetki yükseltmemiz gerekiyor. “sudo -l” komutunu kullandıktan sonra “find” komutunu herhangi bir kullanıcı yetkisiyle çalıştırabileceğimi öğrendim.

![](6.webp){: width="1200" height="500" }

Haliyle root yetkisine erişebilmek için bu komutu manipüle etmem gerekiyordu. Bu yüzden [gtfobins](https://gtfobins.github.io/gtfobins/find) isimli siteyi kullanarak nasıl manipüle edebileceğimi öğrendim ve uyguladım.

![](7.webp){: width="1200" height="500" }
