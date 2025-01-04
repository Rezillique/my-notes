---
title: 5. Nerede Kalmıştık?
draft: false
tags:
  - blog
  - snake
date: 2025-01-04
---
2 günlük kafa tatili ardından tekrar deli deli işler yapmaya devam. Öncelikle bu yazıyı okurken arkada https://open.spotify.com/track/7MFsLdEb5V395J9Zsy9xls?si=64a78d8eae3c455c şu parçayı dinleyin. Spesifik bir sebebi yok kod yazarken arkada (bugünlük) bu tarz şarkılar çalıyor. 

Bugün bir snake oyunu yazacağız. İlk olarak hangi dilde yazacağımı seçmem lazım seçeneklerimiz: Java, Js, Go, Python. Kendime olan saygımdan dolayı Js kullanmıyorum mümkün olduğunca zaten bu blogu oluştururken yeteri kadar Js ve Ts ile uğraştım. Ts kodu okumak benim için zorlu bir süreç.

![[621.png]]

O yüzden seçenekler go, java ve python. Go ileride yapacağım eklemeler ve değişiklikler için doğru tercih olmayabilir bu yüzden bir noktada Pythonda yeniden yazabilirim. Java OOP için ne kadar iyi olsa da sonrası için Java deneyimsizliğimle uğraşmamak için atlıyorum.

Hemen masaüstünde bir Snek klasörü içine bir main.go dosyası ve Vs Code'da go uzantısını indirip her şey tıkırında mı diye main.go dosyası içine 'Hello World' yazıyorum.

``` Go
package main
import "fmt"
func main() {
fmt.Println("Hello World!")
}
```

fmt paketi her teorik dil öğrenmenizde ilk olarak öğrendiğiniz scanf, printf gibi komutları içeren bir paket çok bir olayı yok kısacası. Terminale sonra 

``` Terminal
go run main.go
```

yazarak herhangi bir sorun olmadan Hello World çıktısını alıyoruz.

Snake oyunumuz 3 ana parçadan oluşacak bunlar sırasıyla main.go , oyun.go ve snek.go.

main.go: Arayüzü ve kullanıcı inputlarını bekleyen kodları içerecek.

oyun.go: Oyundaki değişkenlerin  güncellemesini yapıp terminale çizecek.

snek.go: Hız ve pozisyon bilgilerini daha doğrusu Snek objesini halleden kısım.

Ayrıca bir GUI ihtiyacımız olduğu için minik bir araştırmayla tcell isimli bir framework ile terminal tabanlı bir GUI kullanacağız.

``` Go
package main

  

import (

    "log"

    "os"

  

    "github.com/gdamore/tcell"

)

  

func main() {

    screen, err := tcell.NewScreen()

  

    if err != nil {

        log.Fatalf("%+v", err)

    }

    if err := screen.Init(); err != nil {

        log.Fatalf("%+v", err)

    }

  

    defStyle := tcell.StyleDefault.Background(tcell.ColorGreen).Foreground(tcell.ColorRed)

    screen.SetStyle(defStyle)

  

    snekBody := SnakeBody{

        X:      5,

        Y:      10,

        Xspeed: 2,

        Yspeed: 0,

    }

  

    game := Game{

        Screen:    screen,

        snekBody: snekBody,

    }

    go game.Run()

    for {

        switch event := game.Screen.PollEvent().(type) {

        case *tcell.EventResize:

            game.Screen.Sync()

        case *tcell.EventKey:

            if event.Key() == tcell.KeyEscape || event.Key() == tcell.KeyCtrlC {

                game.Screen.Fini()

                os.Exit(0)

            } else if event.Key() == tcell.KeyUp {

                game.snekBody.ChangeDir(-1, 0)

            } else if event.Key() == tcell.KeyDown {

                game.snekBody.ChangeDir(1, 0)

            } else if event.Key() == tcell.KeyLeft {

                game.snekBody.ChangeDir(0, -1)

            } else if event.Key() == tcell.KeyRight {

                game.snekBody.ChangeDir(0, 1)

            }

        }

    }

} 
```

Kod gördüğünüz gibi basit ancak şöyle bir sıkıntı var. BUNU NASIL OKUYACAKSINIZ. Açıklamasız yazdım ve inanılmaz boşluklu duruyor umarım bunu siteye yüklediğimde daha okunaklı olur yoksa okurken başınıza kötü şeyler gelmez. Kısaca ne yaptığımı anlatmam gerekirse İlk önce bir ekran çıkarttık. Sonrasında iki obje tanımladık "snekBody" ve "game" isimli. Bir loop başlattık ve kontrolleri ayarladık. Basit.

Şimdi yeni bir game.go dosyası açıp snek hareketlerini düzenliyoruz. Burada set time sleep koyma sebebimiz daha akıcı bir görüntüye sahip olmak (isterseniz silin başınıza hoş şeyler gelir belki).

``` Go
package main

  

import (

    "time"

  

    "github.com/gdamore/tcell"

)

  

type Game struct {

    Screen   tcell.Screen

    snekBody SnakeBody

}

  

func (g *Game) Run() {

  

    defStyle := tcell.StyleDefault.Background(tcell.ColorBlack).Foreground(tcell.ColorWhite)

    g.Screen.SetStyle(defStyle)

    width, height := g.Screen.Size()

    snakeStyle := tcell.StyleDefault.Background(tcell.ColorWhite).Foreground(tcell.ColorWhite)

  

    for {

        g.Screen.Clear()

        g.snekBody.Update(width, height)

        g.Screen.SetContent(g.snekBody.X, g.snekBody.Y, ' ', nil, snakeStyle)

        time.Sleep(40 * time.Millisecond)

        g.Screen.Show()

    }
  
}
```


Şimdi sıra snek hareketinde. Karmaşık kısım burası çünkü yön değişimi, şimdi bulunduğu lokasyon bilgisinin güncellenmesini istiyoruz. Aynı zamanda bir kenardan girip öteki kenardan çıkabilmesi lazım.

``` Go
package main

  

type SnakeBody struct {

    X      int

    Y      int

    Xspeed int

    Yspeed int

}

  

func (sb *SnakeBody) ChangeDir(vertical int, horizontal int) {

    sb.Yspeed = vertical

    sb.Xspeed = horizontal

}

  

func (sb *SnakeBody) Update(width int, height int) {

    sb.X = (sb.X + sb.Xspeed) % width

    if sb.X < 0 {

        sb.X += width

    }

    sb.Y = (sb.Y + sb.Yspeed) % height

    if sb.Y < 0 {

        sb.Y += height

    }

}
```

Şimdi minik bir çalışma testi yapalım. 

https://youtu.be/OWii6YBxSfM

Linkinden terminalde hız iblisini görebilirsiniz. Biraz yavaşlatırsak iyi olacak. Bunu yapmak için main.go dosyasında Xspeed : 2 satırındaki 2'yi 1 yaparak başarabiliriz.

Teknik olarak bir snek sahibi değiliz. Bir blok var ve hareket ediyor (hala biraz hızlı hareket ediyor). Sıra onu takip edecek blokları eklemekte. Aslında yapmaya çalıştığımız birden fazla objenin hareketini sağlamak ve bunu baş ve kuyruk bloklarına uygun yapması. Normalde bunu queue data structure ile yapabiliriz. Bu da FIFO (first in, first out) mantığıyla çalışan bir sistemle gerçekleşir. 
![[Screenshot 2025-01-04 151939.png]]

Görseldeki gibi bir sistemden bahsediyoruz kısacası. Problem şu ki go queue içeren bir dil değil. Bunu da go'da bulunan slices ile yapıyoruz. Slice dinamik boyutlu array gibi düşünülebilir. 


``` Go
snekParts := []SnakePart{

        {

            X: 5,

            Y: 10,

        },

        {

            X: 6,

            Y: 10,

        },

        {

            X: 7,

            Y: 10,

        },

    }
    ```


main.go kısmına  tam olarak screen.SetStyle(defStyle) satırının hemen altına bunu ekleyip ilk başta yazdığımız X:5 ve Y:10 kısmını siliyoruz. 

oyun.go kısmına ise 

``` Go
func drawParts(s tcell.Screen, parts []SnakePart, style tcell.Style) {

    for _, part := range parts {

        s.SetContent(part.X, part.Y, ' ', nil, style)

    }
    ```

Snekin geri kalan parçalarını çizdirecek komutu ekliyoruz ve ilk yazdığımız 
 ``` Go
 g.Screen.SetContent(g.snekBody.X, g.snekBody.Y, ' ', nil, snakeStyle)
 ```
Kısmını silip 

 ``` Go 
 drawParts(g.Screen, g.snakeBody.Parts, snakeStyle)
 ``` 

Satırını ekliyoruz.

Sonra snek.go dosyasına gidip bir update fonksiyonu eklememiz lazım. Queue sistemini böylelikle taklit etmiş olacağız. 
Burada değişiklikleri yazmaya üşendiğim için kodun son halini atıyorum:

 ``` Go 
 package main

  

type SnakePart struct {

    X int

    Y int

}

  

type SnakeBody struct {

    Parts  []SnakePart

    Xspeed int

    Yspeed int

}

  

func (sb *SnakeBody) ChangeDir(vertical int, horizontal int) {

    sb.Yspeed = vertical

    sb.Xspeed = horizontal

}

  

func (sb *SnakeBody) Update(width int, height int) {

    sb.Parts = append(sb.Parts, sb.Parts[len(sb.Parts)-1].GetUpdatedPart(sb, width, height))

    sb.Parts = sb.Parts[1:]

}

  

func (sp *SnakePart) GetUpdatedPart(sb *SnakeBody, width int, height int) SnakePart {

    newPart := *sp

    newPart.X = (newPart.X + sb.Xspeed) % width

    if newPart.X < 0 {

        newPart.X += width

    }

    newPart.Y = (newPart.Y + sb.Yspeed) % height

    if newPart.Y < 0 {

        newPart.Y += height

    }

    return newPart

}
```

Sonradan eklenen not: Burada kendim yazarken bazı minik hatalar yapmış olacağım ki vs code kızdı. Biraz internetten araştırdıktan sonra daha temiz bir kod bularak onu koydum umarım patlamayız. 

Şimdi test zamanı...

Evet mutlu olmadı. 
Öncelikle bir satırda "}" koymayı unuttuğum için kızdı. Diğer bir sorunsa main.go kısmında hızı yazarken parçaların hızlarını içerecek olan "Parts: snakeParts," satırını yazmayı unuttum ki ilk onu fark ettim. Şimdi tekrar deniyorum...

Çalıştı. Dışarıdan kod ekledim ve çalıştı. Müthiş ve şaşırtıcı. 

Uzun uğraşlar sonucu oyunu tamamlamış bulunmaktayım. Daha okunaklı olsun diye bazı yerlerde düzenlemeler ve yeniden yazmalar (kopyaladım bazı yerleri tabii) sonucunda çalışan bir snek oyunum var. Yarın kodları açıklamalı halde paylaşmayı düşünüyorum.

Kimi zaman hevesle başlanan projelerin bir süre sonra can sıktığını bir yük gibi geldiğini hissetmek normaldir. Basit basit hataları çözmek uzun zaman alır. Bir sürünceme başlar ve sonrasında yeni bir proje fikri aklına gelir veya karşına çıkar. Bir heves ona da başlarsın farklı olacağına inanarak. İlk projen de öyleydi ve yarım bıraktın. Önemli olan o süreci atlatıp işi bitirebilmek ve vazgeçmemek. Önemli olan havai fişekleri izlemek değil her şey kötüye gittiğinde etraftan çıt çıkmadığında ayakta kalabilmek. Bu yazının programlamayla bir bağı yoktur.

AY AY AY AY
AY AY MI AMOR