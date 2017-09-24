## İçeriği Güncelleme
Şu anda veri tabanı bağlamında çok bir değişiklik yok

**`Data/ApplicationDbContext.cs`**

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        // Customize the ASP.NET Identity model and override the defaults if needed.
        // For example, you can rename the ASP.NET Identity table names and more.
        // Add your customizations after calling base.OnModelCreating(builder);
    }
}
```
`ApplicationDbContext`'ine aşağıdaki gibi `DbSet` ekleyin.
Add a `DbSet` property to the `ApplicationDbContext`, right below the constructor:

```csharp
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : base(options)
{
}

public DbSet<TodoItem> Items { get; set; }

// ...
```
`DbSet` veri tabanında bir tablo veya koleksiyonu ifade eder. `Items` adında bir `DbSet<TodoItem>` oluşturarak Entity Framework Core'a `Items` adında bir tabloya `TodoItem` modellerini saklamak istediğinizi söylüyorsunuz. 

Tüm yapmanız gereken bu kadar. Fakat küçük bir problem var. Şu anda veri tabanı ve `ApplicationDbContext` bir biriyle senkronize değil. ( Sadece kodu değiştirmek veri tabanını değiştirmemektedir.)

Veri tabanının yapılan değişikliği uygulayabilmesi için **migration(Göç)** oluşturmamız gerekmektedir.
