---
title: 'HackTheBox Challenges | Flag Command WriteUp'
author: appdone
categories: [HackTheBox Challenges]
render_with_liquid: false
media_subpath: /images/hackthebox_challenges_flag_command/
image:
  path: 1.webp
---

Bu yazımda HackTheBox platformunda bulunan “Flag Command” isimli meydan okumanın çözümünü göstereceğim. Çözümü çok kolay olmakla birlikte küçük bir hikayesi de bulunuyor.

Arkadaşlarımızla dışarıya çıktığımız sırada onlardan koparak kendimizi ormanın en ücra köşesinde buluyoruz. Ağaçların gün ışığını engellediği bu ormanda yönümüzü bulmaya çalışırken çalılıkların ardında insanı andıran sıska bir varlık olduğunu fark ediyoruz. Bu varlık, yorgun düşmemizi bekliyormuş gibi hafifçe sırıtarak bizi izliyor. Amacımız, hava daha fazla kararmadan yönümüzü bulup bu ormandan çıkmaktır.

![](1.webp){: width="1200" height="600" }

Elimizde bir komut satırı var. Start komutunu kullanarak oyuna başlayabiliriz ama önce sayfanın kaynak kodlarına bakmak istedim ve oyunun javascript ile yapıldığını öğrendim.

![](2.webp){: width="1200" height="600" }

Bu durumda oyunu oynamadan sadece javascript kodlarını okuyarak çözebiliriz. Kodları anlayabilmek için herhangi bir programlama dilini bilmek yeterli olacaktır.

```js
import { startCommander, enterKey, userTextInput } from "/static/terminal/js/main.js";
    startCommander();

    window.addEventListener("keyup", enterKey);

    // event listener for clicking on the terminal
    document.addEventListener("click", function () {
      userTextInput.focus();
    });
```

Öncelikle main.js dosyasındaki startCommander() fonksiyonunun çalıştırıldığını görüyoruz. Bu fonksiyonda da fetchOptions() fonksiyonunu çalıştırılıyor.

```js
export const startCommander = async () => {
    await fetchOptions();
    userTextInput.value = "";
    commandText.innerHTML = userTextInput.value;

    await displayLinesInTerminal({ lines: INFO });

    userTextInput.focus();
};
```

fetchOptions() fonksiyonu ise /api/options adresine GET methoduyla bir HTTP isteği gönderiyor.

```js
const fetchOptions = () => {
    fetch('/api/options')
        .then((data) => data.json())
        .then((res) => {
            availableOptions = res.allPossibleCommands;

        })
        .catch(() => {
            availableOptions = undefined;
        })
}
```

/api/options adresine gittiğimde daha fazla komut ile karşılaşıyorum.

![](3.webp){: width="1200" height="600" }

Burada “Blip-blop, in a pickle with a hiccup! Shmiggity-shmack” komutu ilgimi çekiyor. Gönderilen komutlar, /api/monitor adresinden teyit edildiği için kaynak koduna bakarak bayrağı bulamayız. Bu yüzden komutu uygulamada denememiz gerekiyor.

Komutu kullanmadan önce “start” komutunu kullanıp oyunu başlatıyorum. Daha sonra ise bahsettiğim komutu girdiğimde bayrağa ulaşıyor ve oyunu kazanıyorum.

![](4.webp){: width="1200" height="600" }

Söylediğim gibi çok basit bir meydan okumaydı. Yazılan kodları okuyamasak bile geliştirici ayarlarını açıp sayfayı yenileseydik aynı şekilde komutu öğrenip kolayca bayrağa ulaşabilirdik.
