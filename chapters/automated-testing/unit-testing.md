## Unit Testi

Unit testi tek metodun veya kısa bir mantıksal akışı test etmek için kullanılır. Sistemin tamamını test etmek yerine, unit testi **sahte** veriler kullanarak gerçek metodun yapması gereken işi doğru yapıp yapmadığını test eder.

Örneğin `TodoController`'in iki bağımlılığı vardır. Bunlar : `ItodoItemService` ve `UserManager`'dır. `TodoItemService` ise `ApplicationDbContext`'e bağımlıdır. ( Şu şekilde **bağlımlılık grafiği**ni çizebiliriz. `TodoController`->`TodoItemService`->`ApplicationDbContext`)

Normalde ASP.Core bağımlılık enjeksiyon sistemi `TodoController` veya `TodoItemService` oluşturulduğundayukarıda bulunan tüm **bağımlılık grafiğini** enjekte eder.

Fakat unit tesi yazarken bu enjeksiyonları elle yapmak gerekmektedir. Bu da mantığı sınıf veya method bazında izole edebileceğiniz anlamına gelir. (Test yazarken yanlışlıkla veri tabanına yazmak istemezsiniz.)

### Test Projesi Oluşturma

Test için genelde ayrı bir proje oluşturulur. Yeni test projesi eskiden oluşturduğunuz ana projenin yanına ( içine değil) oluşturulur.

Eğer şu anda projenizin ana dizinindeyseniz `cd` ile bir üst klasöre çıkın. Sonra aşağıdaki komutları uygulayarak yeni bir test projesi oluşturun.

```
mkdir AspNetCoreTodo.UnitTests
cd AspNetCoreTodo.UnitTests
dotnet new xunit
```

xUnit.NET unit ve entegrasyon testi için kullanılan popüler bir test iskelettir. Her projemizde olduğu gibi bu proje de aslında NuGet paketlerinden oluşur. `dotnet new xunit` test için gerekli olan herşeyi sizin için oluşturur.

Klasör yapınız şu anda aşağıdaki gibi olmalı:

```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj
```

Test projesinde kullanacağınız sınıflar ana projede olduğundan dolayı bunları ana projeye atfetmeniz gerekmekte.

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```

Ardından `UnitTest1.cs` dosyasınız silin. Artık test yazmaya hazırsınız.

### Servis testi Yazma

`TodoItemService` içindeki `AddItemAsync` metodunun mantığına bakın:

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

    _context.Items.Add(entity);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```
Bu metod gelen verilere bakarak yeni bir nesne üretir ve bunu veri tabanına yazar.

* `OwnerId` giriş yapmış kullanının ID'si olmalı.
* Yeni madde her zaman başlangıçta tamamlanmamış olmalı. (`IsDone = false`)
* Başlık gelen objeden kopyalanmalı : `newItem.Title`
* Her yeni gelen madde bitiş zamanı(`DueAt`) 3 gün sonra olarak ayarlanmalı.

> Bu tipte alınan karalara **iş mantığı(Business Logic)** denir. Tabi bunun yanında **iş mantığı** hesaplamaları da içerir.

Bu karalar mantığın oluşmasına yardımcı olur. Örneğin başka birisi `AddItemAsync` içerisinde değişiklik yapıyor fakat daha önceki kararlardan haberi yok. Bu basit servisler için problem oluşturmayabilir. Fakat büyük servislerde bunun problem oluşturduğunu söylenebilir.

`TodoItemService` metodunu test eden sınıfı yeni oluşturduğunuz projede şu şekilde oluşturun: 

**`AspNetCoreTodo.UnitTests/TodoItemServiceShould.cs`**

```csharp
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using AspNetCoreTodo.Services;
using Microsoft.EntityFrameworkCore;
using Xunit;

namespace AspNetCoreTodo.UnitTests
{
    public class TodoItemServiceShould
    {
        [Fact]
        public async Task AddNewItem()
        {
            // ...
        }
    }
}
```

`[Fact]` özelliği xUnit.NET paketinden gelmektedir, ve bu özelliği ekleyerek bu metodun **test** metodu olduğunu belirtmiş oldunuz.

> Testleri isimlendirmek için bir çok yöntem bulunmaktadır. Hepsinin pozitif ve negatif yanları vardır. Bu projede sondan ekleme ile test sınıflarını oluşturacaksınız. Örnek `TodoItemServiceShould`, elbette siz istediğiniz yöntemi kullanabilirsiniz.

`TodoItemService` `ApplicationDbContext`'e bağımlıdır, yani canlıda bulunan veri tabanına. Bunu test için kullanmak doğru bir yöntem değildir. Bunun yerine Entity İskeletinde bulunan in-memory veri tabanı sağlayıcısını kullanabilirsiniz. Tüm veri tabanı hafıza'da yer aldığından. Test her başladığında temizlenir. Fakat `TodoItemService` bunun farkını anlamayacaktır.

`DbContextOptionsBuilder` ile in-memory veri tabanı oluşturup sonrasında `AddItem`'a çağrı yapabilirsiniz.

```csharp
var options = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseInMemoryDatabase(databaseName: "Test_AddNewItem").Options;

// Set up a context (connection to the DB) for writing
using (var inMemoryContext = new ApplicationDbContext(options))
{
    var service = new TodoItemService(inMemoryContext);

    var fakeUser = new ApplicationUser
    {
        Id = "fake-000",
        UserName = "fake@fake"
    };

    await service.AddItemAsync(new NewTodoItem { Title = "Testing?" }, fakeUser);
}
```

The last line creates a new to-do item called `Testing?`, and tells the service to save it to the (in-memory) database. To verify that the business logic ran correctly, retrieve the item:

```csharp
// Use a separate context to read the data back from the DB
using (var inMemoryContext = new ApplicationDbContext(options))
{
    Assert.Equal(1, await inMemoryContext.Items.CountAsync());
    
    var item = await inMemoryContext.Items.FirstAsync();
    Assert.Equal("Testing?", item.Title);
    Assert.Equal(false, item.IsDone);
    Assert.True(DateTimeOffset.Now.AddDays(3) - item.DueAt < TimeSpan.FromSeconds(1));
}
```

The first verification step is a sanity check: there should never be more than one item saved to the in-memory database. Assuming that's true, the test retrieves the saved item with `FirstAsync` and then asserts that the properties are set to the expected values.

Asserting a datetime value is a little tricky, since comparing two dates for equality will fail if even the millisecond components are different. Instead, the test checks that the `DueAt` value is less than a second away from the expected value.

> Both unit and integration tests typically follow the AAA (Arrange-Act-Assert) pattern: objects and data are set up first, then some action is performed, and finally the test checks (asserts) that the expected behavior occurred.

Here's the final version of the `AddNewItem` test:

**`AspNetCoreTodo.UnitTests/TodoItemServiceShould.cs`**

```csharp
public class TodoItemServiceShould
{
    [Fact]
    public async Task AddNewItem()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: "Test_AddNewItem")
            .Options;

        // Set up a context (connection to the DB) for writing
        using (var inMemoryContext = new ApplicationDbContext(options))
        {
            var service = new TodoItemService(inMemoryContext);
            await service.AddItemAsync(new NewTodoItem { Title = "Testing?" }, null);
        }

        // Use a separate context to read the data back from the DB
        using (var inMemoryContext = new ApplicationDbContext(options))
        {
            Assert.Equal(1, await inMemoryContext.Items.CountAsync());
            
            var item = await inMemoryContext.Items.FirstAsync();
            Assert.Equal("Testing?", item.Title);
            Assert.Equal(false, item.IsDone);
            Assert.True(DateTimeOffset.Now.AddDays(3) - item.DueAt < TimeSpan.FromSeconds(1));
        }
    }
}
```

### Run the test

On the terminal, run this command (make sure you're still in the `AspNetCoreTodo.UnitTests` directory):

```
dotnet test
```

The `test` command scans the current project for tests (marked with `[Fact]` attributes in this case), and runs all the tests it finds. You'll see an output similar to:

```
Starting test execution, please wait...
[xUnit.net 00:00:00.7595476]   Discovering: AspNetCoreTodo.UnitTests
[xUnit.net 00:00:00.8511683]   Discovered:  AspNetCoreTodo.UnitTests
[xUnit.net 00:00:00.9222450]   Starting:    AspNetCoreTodo.UnitTests
[xUnit.net 00:00:01.3862430]   Finished:    AspNetCoreTodo.UnitTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 1.9074 Seconds
```

You now have one test providing test coverage of the `TodoItemService`. As an extra-credit challenge, try writing unit tests that ensure:

* `MarkDoneAsync` returns false if it's passed an ID that doesn't exist
* `MarkDoneAsync` returns true when it makes a valid item as complete
* `GetIncompleteItemsAsync` returns only the items owned by a particular user
