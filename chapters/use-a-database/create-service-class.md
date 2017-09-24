## Yeni Servis Sınıfı Oluşturma

**MVC Temelleri** bölümünde `FakeTodoItemService` adında bir sınıf oluşturup doğrudan yapılacaklar listesi eklemiştik. Artık veri tabanı kaynağına sahip olduğumuzadan dolayı Entity Framework Core ile gerçek verileri kullanabiliriz.

`FakeTodoItemsService.cs`'yi silin ve `TodoItemService.cs` adında yeni bir dosya oluşturun.

**`Services/TodoItemService.cs`**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Services
{
    public class TodoItemService : ITodoItemService
    {
        private readonly ApplicationDbContext _context;

        public TodoItemService(ApplicationDbContext context)
        {
            _context = context;
        }

        public async Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync()
        {
            var items = await _context.Items
                .Where(x => x.IsDone == false)
                .ToArrayAsync();

            return items;
        }
    }
}
```

Dikkat ederseniz **MVC Temelleri** Bölümünde oluşturduğumuz bağımlı enjeksiyon burada da var. (Constructor'a içerik ekleme) Fakat bu defa `ApplicationDbContext` servise enjekte edilmiştir. Daha öncesinde `ApplicationDbContext` servis konteynerına eklendiğinden dolayı burada servise enjekte edilebilir. Aksi halde `ConfigureServices` içerisinde bunu belirtmemiz gerekirdi.

`GetIncompleteItemAsync` metodunu daha yakından inceleyecek olursak. Öncelikle `DbSet` içerisindeki tüm verilere ulaşabilmek için `Items` özelliğini(property) kullanmaktadır. 

```csharp
var items = await _context.Items
```

Ardından, `Where` metodu ile bunlardan sadece tamamlanmamış olanları seçmektedir.


```csharp
.Where(x => x.IsDone == false)
```

`Where` metodu C# LINQ'e ait bir metoddur. Fonksiyonel programlamadan bazı esintiler almıştır. LINQ bizim daha kolay veri tabanı sorguları yazmamıza yardımcı olur. Aslında yukarıda oluşturduğumuz sorgu şu şekildedir: `SELECT * FROM Items where IsDone=0` veya bunun eşiti NoSQL sorgusudur.

Sonunda `ToArrayAsync` metodu bu verileri asenkron dizi haline getirmeye yarar. Yani bu metodun `await` kelimesiyle çağırılması gerekmektedir. 

Metodu daha kusa yazmak için `items`değişkenini silebilirsiniz. Böylece gelen değeri doğrudan değişken olmadan dönderebilirsiniz.


```csharp
public async Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync()
{
    return await _context.Items
        .Where(x => x.IsDone == false)
        .ToArrayAsync();
}
```

### Servis Konteynerını Güncelleme

`FakeTodoItemService`'i sildiğimizden dolayı, `ConfigureServices`'i güncellememiz gerekmektedir.


```csharp
services.AddScoped<ITodoItemService, TodoItemService>();
```
`TodoController` `ITodoItemService`'e bağımlıdır. Bu değişiklikten hiç haberi olmayacaktır. Fakat arka planda sahte servisten gerçek servise geçmiş bulunmaktayız.

### Test

Uygulamayı tekrar çalıştırın ve `http://localhost:5000/todo`'a gidin. Sahte verilerin gittiğini göreceksiniz. Artık uygulamanız veri tabanından gerçek verileri çekmektedir. 

Bir sonraki bölümde uygulamaya yeni yapılacka ekleme gibi özellikler ekleyeceksiniz.
