## Veri tabanına bağlanma

Entity Framework Core kullanarak veri tabanına bağlanmak için bilmeniz gereken birkaç şey var. `dotnet new` kullanarak yeni bir MVC + kişi doğrulama projesi oluşturduğunuzdan dolayı; 


* **Entity Framework Core paketleri**. Bu paketler varsayılan olarak ASP.NET Core projesinde bulunur.

* **Veri tabanı** (doğal olarak).`dotnet new` komutu ile kurulan projenizde ayrıca `app.db` adında bir dosya oluşturuldu. Bu küçük bir SQLite veri tabanıdır. SQLite, başka birşey kurma olmaksızın kullanabileceğiniz küçük bir veri tabanıdır. Böylece prototip için veya küçük projeler için ayrıca bir veri tabanına gerek kalmaz.

* **Veri tabanı kaynak(context) sınıfı**. Veri tabanına bağlanmak için bir başlangıç noktası sınıfı oluşturulur. Burada veri tabanı ile nasıl iletişime geçileceği, okunacağı veya yazılacağı belirlenir. Bu sınıf `Data/ApplicationDbContext.cs` dosyası içerisinde bulunur.

* **Bağlantı Yolu**. İsterseniz kendi bilgisayarınızda veya ağ üzerindeki başka bir bilgisayarda bulunan veri tabanına bağlanmak için bir yol belirlenemeniz gerekmektedir. Şu anda varsayılan bir veri tabanı kullandığımızdan dolayı bunlar otomatik olara `appsettings.json` içerisine yazılmış durumda. Dikkat ederseniz `DataSource=app.db` olarak gözünüze çarpacaktır.

Entity Framework Core veri tabanını bağlantı yolu ile kullanarak veri tabanına bağlanır. Bundan dolayı Entity Framework Core'a hangi kaynağı(context), hangi bağlantı yolunu kullanmak istediğinizi söylemelisiniz. Bunu `ConfigureServices` metodundan değiştirebilirsiniz. Bu metod `Startup` sınıfında bulunur. Bu halihazırda sizin için oluşturulmuş kod.

```csharp
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
```

Bu kod ile; `ApplicationDbContext` service konteynerına eklenir ve Entity Framework Core'a SQLite veri tabanına `appsettings.json`'da bulunan bağlantı yolunu kullanarak bağlanır. 

Gördüğünüz gibi `dotnet new` sizin için birçok şeyi hazır hale getiriyor. Veri tabanımız kuruldu ve artık kullanılmaya hazır. Fakat şu anda hiç bir tablomuz bulunmamakta. `TodoItem` modellerini tutmak için kaynağı(context) güncellemeli ve veri tabanını taşımalıyız(migrate).

