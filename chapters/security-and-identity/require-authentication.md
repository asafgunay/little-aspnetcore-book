## Kimlik Doğrulama
Çoğu uygulamada kullanıcı girişi mevcuttur. Kullanıcı giriş yaparak kendine ait bölümleri gezebilir veya işlem yapabilir. Ana sayfayı herkese göstermek mantıklı olabilir, fakat yapılacaklar listesini sadece giriş yapmış olan kişilerin görmesi gerekmekte.

ASP.NET Core `[Authorize]` özelliğini kullanarak kullanıcının bazı aksiyon veya kontrolörün tamamına giriş kontrolü ekleyebiliriz. Aşağıdaki kimlik kontrolü bu kontrolör içerisindeki tüm aksiyonlara uygulanır.


```csharp
[Authorize]
public class TodoController : Controller
{
    // ...
}
```

Aşağıdaki `using` cümlesini de sınıfınız üzt tarafına yapıştırın.
```csharp
using Microsoft.AspNetCore.Authorization;
```
Uygulamanızı çalıştırıp kullanıcı girişi yapmadan `/todo` sayfasına gidin. Giriş sayfasına otomatik olarak yönlendirildiğinizi göreceksiniz.

> Doğrulama ve Yetki birbiri ile karıştırılan özellikler. Burada doğrulama yapıyorsunuz. Kullanıcının neye yetkisi olduğunu belirtmedik. Yani sadece giriş yapıp yapmadığı bilgisine bakıyorsunuz.

