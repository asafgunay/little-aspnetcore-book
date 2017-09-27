## Uygulama İçerisinde Kişiye Has Özellikleri Kullanma

Yapılacaklar listesinde hala kullanıcı bazında bir ayrım bulunmamaktadır.`[Authorize]` özelliği sayesinde kullanıcı giriş yaptıysa tüm listeyi görebiliri. Bunun ile birlikte giren kullanıcıya göre bu filtreyi genişletmek gerekmekte.

Öncelikle `UserManager<Applicationuser>` 'ı `TodoController`a enjekte edin.

**`Controllers/TodoController.cs`**

```csharp
[Authorize]
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;
    private readonly UserManager<ApplicationUser> _userManager;

    public TodoController(ITodoItemService todoItemService,
        UserManager<ApplicationUser> userManager)
    {
        _todoItemService = todoItemService;
        _userManager = userManager;
    }

    // ...
}
```
Üst taraftaki `using` cümlesine dikkat edin.

```csharp
using Microsoft.AspNetCore.Identity;
```
`UserManager` sınıfı ASP.NET Core Kimlik'e ait bir sınıftır. Bu sınıf sayesinde kullanıcıya ait bilgileri alabilirsiniz.

```csharp
public async Task<IActionResult> Index()
{
    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var todoItems = await _todoItemService.GetIncompleteItemsAsync(currentUser);

    var model = new TodoViewModel()
    {
        Items = todoItems
    };

    return View(model);
}
```
Aksiyon içerisinden `User` özelliğini kullanarak `UserManager` vasıtası ile kullanıcıların özelliklerine erişebilirsiniz.


```csharp
var currentUser = await _userManager.GetUserAsync(User);
```
Şimdi sorabilirsiniz:"Madem `User` objemiz zaten aksiyondan erişilebilir durumda, peki neden `UserManager`'a çağrı yapmamız gerekiyor". Burada bulunan `User` objesinin içerisinde çok basit veriler tutulur.`UserManager` ise bu kullanıcıya ait tim bilgilerin veri tabanından getirilmesini sağlar.

`currentUser` değeri hiç bir zaman null olmamalıdır. Zaten daha önce `[Authorize]` ile kontrolörü ile giriş yapılmadan kullanılamayacağını belirtmiştiniz. Fakat yine de eğer kullanıcı giriş yapmadıysa veya her hangi bir şekilde o an bir problem olduysa onu giriş sayfasına yönlendirmek iyi bir pratiktir. Bunun için `Challenge()` metodunu kullanabilirsiniz. 

```csharp
if (currentUser == null) return Challenge();
```
Artık `ApplicationUser` parametresini `GetIncompleteItemsAsync`'e gönderdiğinize göre, `ITodoItemService` arayüzünde de değişiklik yapmalısınız.


**`Services/ITodoItemService.cs`**

```csharp
public interface ITodoItemService
{
    Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync(ApplicationUser user);
    
    // ...
}
```
Sıradaki işlem veri tabanını güncelleyerek sadece giriş yapan kullanıcıya has verileri getirmektir.

### Veri tabanını güncelleme

`TodoItem` modeline yeni bir özellik ekleyerek kullanıcıların bilgilerinin tutulmasını sağlamalısınız.

```csharp
public string OwnerId { get; set; }
```
Tabi şu anda sadece modelde değişiklik yaptınız ve bunu henüz uygulamadınız. Bundan dolayı daha önce yaptığımız gibi **göç** ettirmeniz gerekmekte. Bunun için terminalde veya komut satırında `dotnet ef` komutunu kullanarak aşağıdaki işlemi yapın.


```
dotnet ef migrations add AddItemOwnerId
```

Bu yeni bir **göç** oluşturarak bunun adını `AddItemOwner` verir. Modelde yaptığımız değişiklikler burada da görülebilir. 

> Eğer SQLite kullanıyorsanız bu **göç**'ü otomatik yapamazsınız. Bunun için **Migration Oluşturma** bölümüne bakabilirsiniz. Tekrar `dotnet ef` komutu ile bu **göç**ü uygulayabilirsiniz.

```
dotnet ef database update
```

### Servis sınıfını güncelleme

Veri tabanı ve veri tabanı konteksi güncellendi, artık `TodoItemService` içerisinde bulunan `GetIncompleteItemsAsync` metodunu değiştirebilir ve `Where` sorgusu ekleyebilirsiniz. 


**`Services/TodoItemService.cs`**

```csharp
public async Task<IEnumerable<TodoItem>> GetIncompleteItemsAsync(ApplicationUser user)
{
    return await _context.Items
        .Where(x => x.IsDone == false && x.OwnerId == user.Id)
        .ToArrayAsync();
}
```
Yeni bir kullanıcı oluşturur ve giriş yaparsanız, yapılacaklar listesini boş göreceksiniz. Malesef yeni bir yapılacak eklediğinizde de bu değişmeyecek, çünkü yeni yapılacak ekleme olayının içerisinde kullanıcı bilgilerini kaydetme işlemini yapmanız henüz.

### Yeni Yapılacak Olayı Güncellemesi ve Tamamlandı Operasyonu

`UserManager` kullanarak giriş yapmış kullanıcının bilgilerini `AddItem` ve `MarkDone` aksiyonları içerisine paslayabilirsiniz. Bunu daha önce `Index` aksiyonunda yapmıştınız. Buradaki tek fark bu metodların kullanıcıyı eğer giriş yapmamışsa giriş ekranına yönlendirmek yerine `401 Unauthorized` döndürmesidir.

`TodoController` içerisini aşağıdaki gibi güncelleyin.


```csharp
public async Task<IActionResult> AddItem(NewTodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Unauthorized();

    var successful = await _todoItemService.AddItemAsync(newItem, currentUser);
    if (!successful)
    {
        return BadRequest(new { error = "Could not add item." });
    }

    return Ok();
}

public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty) return BadRequest();

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Unauthorized();

    var successful = await _todoItemService.MarkDoneAsync(id, currentUser);
    if (!successful) return BadRequest();

    return Ok();
}
```
İki serviste `ApplicationUser` parametresini kabul etmeli. `ITodoItemService` içerisini aşağıdaki gibi güncelleyin.

```csharp
Task<bool> AddItemAsync(NewTodoItem newItem, ApplicationUser user);

Task<bool> MarkDoneAsync(Guid id, ApplicationUser user);
```

Son olarak `TodoItemService` içerisinde bulunan servis metodunu güncelleyin.
`AddItemAsync` metodu için `Owner` özelliğini yeti `TodoItem` oluştururken ayarlayın.

```csharp
public async Task<bool> AddItemAsync(NewTodoItem newItem, ApplicationUser user)
{
    var entity = new TodoItem
    {
        Id = Guid.NewGuid(),
        OwnerId = user.Id,
        IsDone = false,
        Title = newItem.Title,
        DueAt = DateTimeOffset.Now.AddDays(3)
    };

    // ...
}
```
`MarkDoneAsync` metodu içerisinde kullanıcıyı kontrol etmelisiniz. Böylece sadece yapılacak ID'si ile güncelleme yapılamayacaktır.

```csharp
public async Task<bool> MarkDoneAsync(Guid id, ApplicationUser user)
{
    var item = await _context.Items
        .Where(x => x.Id == id && x.OwnerId == user.Id)
        .SingleOrDefaultAsync();

    // ...
}
```
Bunu da tamamladık. Bundan sonra her kullanıcı kendi adına yapılacak ekleyebilecek ve başkası bunu göremeyecek veya tamamlandı olarak belirtemeyecek.