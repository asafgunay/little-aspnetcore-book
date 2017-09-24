## Migration(Göç) Oluşturma

Migration(Göç) veri tabanı yapısındaki değişklikleri tutmaya yarar. Böylece veri tabanı üzerinde geriye dönüç(roll back) veya aynı yapıya sahip ikinci bir veri tabanı yapmak mümkündür. Böylece değişen her kolon'u takip edebilmek mümkündür.

Kaynak(Context) ve veri tabanı şu anda senkronize olmadığından yeni bir **Göç** oluşturarak veri tabanını güncellememiz gerekmektedir. Bunu `Items` alanı için yapacağız.

```
dotnet ef migrations add AddItems
```

Bu **Kaynak** ta yaptığınız değişiklikleri inceleyerek `AddItems` adında yeni bir **Göç** oluşturur


> Eğer şöyle bir hata alıyorsanız : 
> `No executable found matching command "dotnet-ef"`
> Doğru klasörde olduğunuza emin olun. Bu komutlar ana klasörden çalıştırılmalıdır. Ana klasör `Program.cs` nin bulunduğu klasördür.

Eğer `Data/Migration` klasörünü açarsanız, bir kaç dosya göreceksiniz.

![Multiple migrations](migrations.png)

Bizim için ilk oluşturulan **göç** dosyası `00_CreateIdentitySchema.cs` adı ile oluşturulmuş ve veri tabanı bu şekilde güncellenmiştir. `AddItems` dosyasının önüne `timestamp` eklenmiş ve bu şekilde **göç** dosyası oluşturulmuştur.

> Tavsiye: **Göç** listesini `dotnet ef migrations list` ile görebilirsiniz.

**Göç dosyasını açtığınızda `Up` ve `Down` adında iki metod göreceksiniz.


**`Data/Migrations/<date>_AddItems.cs`**

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // (... some code)

    migrationBuilder.CreateTable(
        name: "Items",
        columns: table => new
        {
            Id = table.Column<Guid>(type: "BLOB", nullable: false),
            DueAt = table.Column<DateTimeOffset>(type: "TEXT", nullable: true),
            IsDone = table.Column<bool>(type: "INTEGER", nullable: false),
            Title = table.Column<string>(type: "TEXT", nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Items", x => x.Id);
        });

    // (some code...)
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    // (... some code)

    migrationBuilder.DropTable(
        name: "Items");

    // (some code...)
}
```
`Up` metodu **Göç**ü çalıştırdığımızda veri tabanına etki edecek metoddur. Kaynağa `DbSet<TodoItem>` eklediğinizden dolayı Entity Framework Core `Items` adında bir tablo oluşturacak ve `TodoItem` modelinde bulunan alanlara uyacak kolonlar oluşturacaktır.

`Down` metodu ise tersini yapar. **Göç** geri alındığında `Items` tablosunu siler.


### SQLite Sınırlılığına Geçici Çözüm

Projede SQLite kullanmanın bazı sınırlılıkları var. Bu sınırlılıklar düzelene kadar şu şekilde geçici bir çözüm yolu kullanabilirsiniz.


* `Up` metodundaki `migrationBuilder.AddForeignKey` satırlarını yorum olarak değiştirin. ( `//` kullanabilirsiniz.)
* `Down` metodundaki `migrationBuilder.DropForeignKey` satırlarını yorum olarak değiştirin.

Eğer MySQL veya SQL Server gibi tam teşekküllü bir SQL veri tabanı kullanıyor olsaydınız böyle bir değişikliğe ihtiyacınız olmayacaktı. **Göç** dosyasını çalıştırdığınızda hiç bir sorun almayacaktınız.

### **Göç**ü uygulayın

**Göç**ü oluşturduktan sonraki son basamak uygulanmasıdır : 

```
dotnet ef database update
```

Yukarıdaki komut Entity Framework Core'un `Items` adında bir tablo oluşturmasına neden olacaktır.

> Eğer bu güncellemeyi geri almak istiyorsanız. Daha önceki bir **göç** ismini vererek bu işlemi gerçekleştirebilirsiniz. 
> `dotnet ef database update CreateIdentitySchema`
> Bu komut `CreateIdentitySchema` dan sonra oluşturulmuş tüm **göç*lerin `Down` metodunu çalıştıracaktır.

> Eğer veri tabanını tamamen silmek istiyorsanız. `dotnet ef database drop` kullanabilirsiniz. Ardından `dotnet ef database update` yazdığınızda son **Göç** e kadar tüm göçler çalıştırılır.

Şu anda veri tabanı ayağını tamamlamış durumdayız. Sonraki adımımız bu kaynağı servis içinde kullanmak.
