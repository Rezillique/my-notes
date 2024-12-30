---
title: 0'dan 100'e Çıkış
draft: false
tags:
  - değişiklik
  - blog
date: 2024-12-30
---
 Snake kullanarak şifreleme?
 
 Kendi QR kodunu oluşturma?
 
 Drive üzerine linux kurma?


3-4 gün blog paylaşma üzerine kafa kırıp, defalarca log okuduktan sonra istediğim sonucu elde edemedim. Her zaman yaptığım gibi 0'dan başlayarak her şeyi yeniden yapmaya başladım. Bu sefer ==Quartz== diye bir uygulama kullandım. Hugo'nun aksine çok daha iyi bir dokümantasyona sahip. Bütün işler tıkırında giderken minik problemlerle karşılaştım. Bunlardan biri bu blogu githubta paylaşabilmem için deploy.yml bir dosyayı 

```
quartz/.github/workflows/deploy.yml
```

adresinde oluşturup düzenlememi gerektiriyor bunu powershell kullanarak nasıl yaparız. Çok basit: 

``` Shell
echo quartz/.github/workflows/deploy.yml
```

AMA NEDENSE BULUNDUĞUM KLASÖRE deploy.yml oluşturup orada düzenlemişim. 15 dakika neden olmadı diye düşünürken şans eseri dosyanın yanlış yerde olduğunu fark ettim.  Sorun hızlıca çözüldü dosya doğru yere taşınarak. İkinci problem bu siteyi github'a pushlarken biraz zaman alıyor. 1-2 dk beklemek yeterli kısacası. Sonrasında 

https://rezillique.github.io/my-notes/

linkiyle içerdeyiz. Bu başarı sağlandıktan sonra farklı ağlardan girilebildiğini test etmek ve mobilde nasıl durduğuna bakmak kaldı. Mobilde güzel durmuyor bir tık ayar lazım ama site çalışıyor yani herkes erişebilir. Sırada siteyi özelleştirmek var. Bunları çeşitli pluginler ve hiç sevmesemde HTML kullanarak yapmak durumundayım. HTML bilgim

``` HTML
<h1>HABU SİTEDUR <h1> 
```

kısmından ibaret yani sadece başlık atabilirim. 


Bütün siteyle alakalı sorunlar bir yana artık çalışan ve aktif güncelleyebildiğim bir site olduğuna göre diğer konulara ağırlık verebilirim.