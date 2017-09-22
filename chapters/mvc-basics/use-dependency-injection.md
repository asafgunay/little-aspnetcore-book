## Bağımlı Enjeksiyon Kullanma
Tekrar `TodoController`'a dönün ve oluşturduğumuz `ITodoItemService`'i kullanmak için aşağıdaki eklemeleri yapın.

```csharp
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;

    public TodoController(ITodoItemService todoItemService)
    {
        _todoItemService = todoItemService;
    }

    public IActionResult Index()
    {
        // Get to-do items from database

        // Put items into a model

        // Pass the view to a model and render
    }
}
```
`ITodoItemService` servis isim uzayında olduğundan dolayı bunu bu sınıfınıza `using` cümlesini sayfanın en üst bölümüne yazarak getirmelisiniz.

```csharp
using AspNetCoreTodo.Services;
```

Yazdığımız ilk satır özel(private) bir değişken tanımlayarak `ITodoItemService` referenasını tutar. Daha sonra bu değişken üzerinden `Index` aksiyonu içerisinde servisimizi kullanabileceğiz.

`public TodoController(ITodoItemService todoItemService)` satırı ise **constructor** tanımıdır. Yani sınıf ilk defa başlatıldığında bu metod bir defa çalışır. Bu sınıf ilk defa çalıştığında servisimizi bu kontrolöre enjekte etmek istediğimizden yukarıdaki gibi değişkenimizi **constructor** içerisinde ayarlıyoruz.

> Arayüzler(Inteface) işimizi çok kolaylaştırmaktadır. Örneğin bu kontrolörümüzde biz `ItodoItemService` arayüzünü kullanıyoruz. Özel bir servise bağlı olmadığımızdan bu servisimizi şu anda `ITodoItemService` olarak tanımladık. İleride bunu değiştirip gerçeğini kullandığımız zaman kontrolör içierisinde hiç bir değişiklik yapmamıza gerek kalmaz. Bu olay ayrıca **Otomatik Test** işini de kolaylaştırır. Bunu ileriki bölümlerde daha kapsamlı göreceğiz.

En nihayetinde `ITodoItemService`'ini aksiyonunuz içerisinde kullanacağız.

```csharp
public IActionResult Index()
{
    var todoItems = await _todoItemService.GetIncompleteItemsAsync();

    // ...
}
```
Hatırlarsanız `GetIncompleteItemsAsync` metodu `Task<IEnumerable<TodoItem>>` döndürmüştü. Geriye `Task` döndüğünde aklınıza ilk gelmesi gereken şey sonuçların doğrudan gelmeyebileceğidir. Bunun için `await` kelimesini kullanıyoruz. Bu kelime sonuna komutumuzun cevap gelene kadar beklemesini sağlıyor.

> Örneğin bu javascript te promise ile çözülmeye çalışıldı fakat daha önce ajax ve callback fonksyionları ile çalışanlar daha iyi hatırlayacaklardır. Bu fonksiyonları kullanmak cehenneme dönüşebilir. ASP.NET core güzel bir yazım tarzı ile bizi bu karmaşıklıktan kurtarmaktadır.

Kodunuzdan veri tabanına çağrı yaptığınızda `Task` kalıbını kullanmak oldukça yaygındır. Çünkü veri tabanından veya ağdan verileri alana kadar bize hiç bir değer döndermeyecektir. Daha önce de bahsettiğimiz gibi `Task` Javascriptte bulunan promise yapısınıa benzemektedir.


> Javascript'e göre `Task` Javascript callback fonksyionlarına göre daha kullanışlıdır. `await` kelimesi işimizi olukça kolaylaştırmaktadır. `await` kodumuzun asenkron çağrılar işin durdurulmasını sağlar. Bu sırada servisimiz verileri veri tabanından alır ve geri geri gönderir. Sonra kodumuz kaldığımız yerden devam eder. Tam olarak anlayamadıysanız ileriki bölümlerde daha iyi anlayacaksınız.

Burada bilmeniz gereken bir şey `IActionResult` yerine `Task<IActionResult>` kullanmanız gerektiği ve bunu asenkron olarak tanımlamak için `async` kelimesi kullanmanız gerektiğidir. 

```csharp
public async Task<IActionResult> Index()
{
    var todoItems = await _todoItemService.GetIncompleteItemsAsync();

    // Put items into a model

    // Pass the view to a model and render
}
```
Nerdeyse sonuna geldik. `TodoController`'ı  `ITodoItemService`'e bağlı hale getirdiniz. Fakat şu anda hala `ITodoItemService` arayüzüne `FakeTodoItemService` kullanması gerektiğini söylemedik. Şimdilik bir sınıfımız var neden böyle birşey yapmaya ihtiyacımız var gibi düşünebilirsiniz. Fakat ileride daha fazla sınıf kullanabiliriz. Kural olarak bunların hepsinin tanımlanması gerekmektedir.

Tanımlamak için  ( Spring framework @wired ) `Startup` sınıfı içerisindeki `ConfigureServices` metodunu kullanacağız. Bu sınıf şimdilik aşağıdaki gibi görünmektedir..

**Startup.cs**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Bazı kodlar

    services.AddMvc();
}
```
ASP.NET Core'a `ITodoItemService` için `FakeTodoItemService` kullan demek için bu metod içerisinde herhangi bir yere aşağıdaki kodu yazmalıyız.


```csharp
services.AddScoped<ITodoItemService, FakeTodoItemService>();
```

`AddScoped` sizin servisinizi **servis konteyner**'ına **scoped** olarak ekler. Bu her `FakeTodoItemService` çağırıldığında yeniden üretilmesini sağlar. Bu veritabanaı ilişkili servislerde çokça kullanılan yöntemdir. Ayrıca servisi **singletons** olarak da tanımlayabilirsiniz. Bu da bu servisin sadece bir defa başlangıçta üretilmesini sağlar.


Bu kadar! Artık `TodoController`'a talep geldiğinde. ASP.NET Core kontrolör `ITodoItemService`'i talep ettiğinde `FakeTodoItemService`'i sunacak. Çünkü biz bağı bulunan servisimizi enjekte ettik. İşte bu kalıba  **dependency injection** denilmektedi. 
