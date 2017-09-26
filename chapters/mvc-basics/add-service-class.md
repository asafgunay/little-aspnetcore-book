## Servis Sınıfı Ekleme

Modeli, Görüntü dosyasını ve kontrolörü oluşturdunuz. Kontrolör içerisinde oluşturduğunuz modeli kullanabilmek için yapılacaklar listesini veri tabanından alacak kodu yazmanız gerekmekte.

Aslında veri tabanına bağlanıp model oluşturma işleminin tamamını kontrolör içerisinde yapabiliriz. Fakat bu iyi bir pratik değildir. Bunun yerine tüm veri tabanı kodlarını **servis** adını verdiğimiz sınıflarda yaparsak kontrolörlerimiz olabildiğince sade olur. Bu da test etmeyi ve daha sonra veri tabanı kodlarında değişiklik yapmayı kolaylaştırır.


> Uygulamayı mantık katmanı, veri tabanı katmanı ve uygulama katmanı gibi katmanlara ayırma olayına bazen **3-ties** veya **n-tier** mimari de denir.

.NET ve C# **interface** konseptine sahiptir. Interface metodların ve özelliklerin aslında işi yapan sınıftan ayrılması olarak tanımlanabilir. Interface sınıflarınızı decoupled(ayrılmış) şekilde tutmanızı sağlar. Bu da test etmeyi daha kolay hale getirir. Bunu daha sonra  **Otomatik Test** bölümünde göreceğiz.

Önce Interface'i oluşturarark yapılacaklar listesi için oluşturulacak servisi tanımlayalım. Bu oluşturacağımız dosyaları da `Services` klasörünün içerisinde oluşturmalıyız.

**`Services/ITodoItemService.cs`**

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo.Services
{
    public interface ITodoItemService
    {
        Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync();
    }
}
```

Dikkat ederseniz dosyanın isimuzayı(namespace)  `AspNetCoreTodo.Services`. İsimuzayı .NET kodlarının organize olmasına yarar. Burada gördüğünüz isimuzayı klasör yapısını takip etmiştir.

Bu dosya `AspNetCoreTodo.Services` isimuzayına sahip ve `TodoItem` sınıfını talep ettiğinden dolayı `AspNetCoreTodo.Models` isimuzayını bu dosyaya `using` cümlesi kullanarak eklememiz gerekmektedir. Aksi halde aşağıdaki gibi bir hata göreceksiniz.

```
The type or namespace name 'TodoItem' could not be found (are you missing a using directive or an assembly reference?)
```

Interface olmasından dolayı aslında çalışan hiç bir ko yoktur sadece tanımı oluşturduk. Yapacağımız Serviste `GetIncompleteItemsAsync` metodunu uygulamamız ve `IEnumerable<TodoItem>` yerine `Task<IEnumerable<TodoItem>>` döndürmemiz gerekmektedir.

> Eğer kafanız karıştıysa şu şekilde düşünebilirsiniz. "Yapılacaklar'ı içeren listeyi içeren bir görev"

`Task` tipi promise ve future benzeri bir tiptir, burada kullanmamızın nedeni bu metodu **asenkron** yapmak istememizden dolayıdır. Diğer bir deyişle veri tabanından alacağımız listenin ne zaman geri döneceğini bilmediğimizden bu şekilde kullanmaktayız.

Artık Interface'i tanımladık. Gerçekten işi yapan servis sınıfını yapmaya başlayabiliriz. Veri tabanı kodunu daha sonraki **Veri tabanı kullanma** bölümünde daha derinine inceleyeceğiz. Şimdilik örnek veriler kullanarak işlemimizi gerçekleştireceğiz.


**`Services/FakeTodoItemService.cs`**

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo.Services
{
    public class FakeTodoItemService : ITodoItemService
    {
        public Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync()
        {
            // Return an array of TodoItems
            IEnumerable<TodoItem> items = new[]
            {
                new TodoItem
                {
                    Title = "Learn ASP.NET Core",
                    DueAt = DateTimeOffset.Now.AddDays(1)
                },
                new TodoItem
                {
                    Title = "Build awesome apps",
                    DueAt = DateTimeOffset.Now.AddDays(2)
                }
            };

            return Task.FromResult(items);
        }
    }
}
```

`FakeTododItemService` `ITodoItemService` arayüzünü sağlamıştır. Şu anda her defasında iki tane yapılacak döndermektedir. Fakat ilerde *Veri Tabanı Kullanma* bölümünde bu verileri veri tabanından çekeceğiz.
