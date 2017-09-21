## Kontrolör Oluşturma

Hali hazırda Controller klasöründe `HomeController` gibi ana sayfayı gösteren bir kontrolör dahil birkaç kontrolör bulunmaktadır. Sunucumuz çalıştığında tarayıcıdan `http://localhost:5000` adresine gittiğimizde bize o sayfayı gösteren kontrolör  `HomeController` dür.

Yapılacaklar listesi için bu klasör içerisine `TodoController` adında bir kontrolör oluşturalım. Sonuna .cs eklentisini koymayı unutmayın. Oluşturduktan sonra içerisine aşağıdaki kodu yapıştırın


**`Controllers/TodoController.cs`**

``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace AspNetCoreTodo.Controllers
{
    public class TodoController : Controller
    {
        // Aksiyonlar buraya gelecek
    }
}
```

Kontrolör tarafından idare edilen rotalara **aksiyon** denir. Bunlar kontrolör içindeki metodlarla ifade edilir. Örneğin `HomeController` üç tane aksiyon bulunmaktadır (`Index`, `About`, ve `Contact`). Bu aksiyonlar ASP.NET Core tarafında şu şekilde rotalara yerleştirilmişlerdir.

```
localhost:5000/Home         -> Index()
localhost:5000/Home/About   -> About()
localhost:5000/Home/Contact -> Contact()
```

ASP.NET Core rotaları yaratmak için birkaç kalıp kullanmaktadır. Örneğin kontrolörümüzün ismi `FooController` olur ise rotamız doğrudan `/Foo` oluyor. `/Foo/Index` ile `/Foo` aynı anlama geliyor ayrıca `Index` aksiyonu yazmanıza gerek kalmıyor yani. Bu proje boyunca biz varsayılan haliyle kullanacağız.

`TodoController` içerisine `// Aksiyon buraya gelecek` yazısı yerine `Index` aksiyonu oluşturalım

```csharp
public class TodoController : Controller
{
    public IActionResult Index()
    {
        // Database'den değerleri getir

        // Gelen değerleri yeni modele koy

        // Modeli Görünyüye ekle ve sayfayı göster.
    }
}
```
Aksiyon metodları Görüntü(view), JSON verisi, veya HTTP status kodu `200 OK` veya `404 Not Found` gibi esnek değerler döndürebilir.

Pratikte kontrolörün olabildiğince basit olmasına özen gösterilir. Böyle bir kontrolörü hedef olarak aldığımızda kontrolör sadece veritabanından değerler alıp bunları modele koyma işini yapar. Ardından önyüze gönderir.

Kontrolörün devamını yazmadan önce Model ve Görüntü (view) oluşturmanız gerekmektedir.
