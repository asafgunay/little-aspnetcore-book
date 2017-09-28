## Role göre yetkilendirm

Yetki ve izin mekanizmasını genel olarak Roller vasıtasıyla gerçekleştirilmektedir. Örneğin Admin roünde bir kişi tüm kullanıcıları görüp bu kullanıcılar üzerinde işlem yapabilirken. Normal kullanıcı rolune sahip birisi sadece kendine ait bilgileri görebilir.

### Yönetici Ekleme

Önce aşağıdaki gibi bir kontrolör oluşturun

**`Controllers/ManageUsersController.cs`**

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Controllers
{
    [Authorize(Roles = "Administrator")]
    public class ManageUsersController : Controller
    {
        private readonly UserManager<ApplicationUser> _userManager;
        
        public ManageUsersController(UserManager<ApplicationUser> userManager)
        {
            _userManager = userManager;
        }

        public async Task<IActionResult> Index()
        {
            var admins = await _userManager
                .GetUsersInRoleAsync("Administrator");

            var everyone = await _userManager.Users
                .ToArrayAsync();

            var model = new ManageUsersViewModel
            {
                Administrators = admins,
                Everyone = everyone
            };

            return View(model);
        }
    }
}
```
Dikkat ettiyseniz `[Authorize]` bölümünde Rol belirtilmiş. Bu kullanıcının hem giriş yaptığını hem de `Administrator` rolüne sahip olduğunu garanti eder. Aksi halde kullanıcı bu kontrolöre giriş yapamaz.

Şimdi bir Görüntü Modeli oluşturun

**`Models/ManageUsersViewModel.cs`**

```csharp
using System.Collections.Generic;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo
{
    public class ManageUsersViewModel
    {
        public IEnumerable<ApplicationUser> Administrators { get; set; }

        public IEnumerable<ApplicationUser> Everyone { get; set; }
    }
}
```

Son olarak Görüntüyü oluşturun 
**`Views/ManageUsers/Index.cshtml`**

```html
@model ManageUsersViewModel

@{
    ViewData["Title"] = "Manage users";
}

<h2>@ViewData["Title"]</h2>

<h3>Administrators</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Administrators)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>

<h3>Everyone</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Everyone)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>
```

Uygulamayı başlatın, normal kullanıcı olarak giriş yapın ve `/ManageUsers` sayfasına gidin. Aşağıdaki gibi bir Giriş Engellendi sayfasını göreceksiniz.
![Access denied error](access-denied.png)

Bu sayfayı görmenizin nedeni web uygulamanıza `Administrator` olarak giriş yapmadığınızdan dolayıdır.


### Test Admin Hesabı Oluşturma


Kullanıcılara kayıt olurken bir onay kutusu ile admin olmak istiyormusunuz demek çokta mantıklı değidir. Bunun yerine `Startup` sınıfına yazacağımız kod ile test admin hesabı oluşturmak mümkündür. `Startup` sistem ayağa kalkarken çalışacak ve böylece bir admin hesabı uygulama başladığında kullanımınıza hazır hale gelecektir.

Aşağıdaki değişikliği `Configure` metodu içinde yer alan `if(env.IsDevelopment())` kontrolünün içerisine yazın.


**`Startup.cs`**

```csharp
if (env.IsDevelopment())
{
    // (... some code)

    // Make sure there's a test admin account
    EnsureRolesAsync(roleManager).Wait();
    EnsureTestAdminAsync(userManager).Wait();
}
```
`EnsureRoleAsync` ve `EnsureTestAdminAsync` metodları `RoleManager` ve `UserManager` servislerine ihtiyaç duyar. Bundan dolayı bu servisleri `Startup` sınıfına enjekte etmemiz gerekmekte. Bunun için aşağıdaki değişikliği yapın

```csharp
public void Configure(IApplicationBuilder app,
    IHostingEnvironment env,
    UserManager<ApplicationUser> userManager,
    RoleManager<IdentityRole> roleManager)
{
    // ...
}
```

Aşağıdaki iki metodu `Configure` metodunun altına yapıştırın
Önce `EnsureRolesAsync` metodu

```csharp
private static async Task EnsureRolesAsync(RoleManager<IdentityRole> roleManager)
{
    var alreadyExists = await roleManager.RoleExistsAsync(Constants.AdministratorRole);
    
    if (alreadyExists) return;

    await roleManager.CreateAsync(new IdentityRole(Constants.AdministratorRole));
}
```
Bu metod veri tabanında `Administrator` rolünün olup olmadığını kontrol eder. Eğer yoksa oluşturur. Her yerde `Administrator`'ü elle yazmaktansa `Constants` adında yeni bir sınıf oluşturup onun içerisine sabit değerlerimizi yazabiliriz.


**`Constants.cs`**

```csharp
namespace AspNetCoreTodo
{
    public static class Constants
    {
        public const string AdministratorRole = "Administrator";
    }
}
```
> Daha önce oluşturduğumuz `ManageUsersController` kontrolörünü de bu sabit ile değiştirebilirsiniz.

`EnsureTestAdminAsync` metodunu yazın

**`Startup.cs`**

```csharp
private static async Task EnsureTestAdminAsync(UserManager<ApplicationUser> userManager)
{
    var testAdmin = await userManager.Users
        .Where(x => x.UserName == "admin@todo.local")
        .SingleOrDefaultAsync();

    if (testAdmin != null) return;

    testAdmin = new ApplicationUser { UserName = "admin@todo.local", Email = "admin@todo.local" };
    await userManager.CreateAsync(testAdmin, "NotSecure123!!");
    await userManager.AddToRoleAsync(testAdmin, Constants.AdministratorRole);
}
```
Eğer `admin@todo.local` isminde bir kullanıcı yoksa, bu metod yeni bir kullanıcı oluşturacak ve geçici şifre atayacak. Giriş yaptıktan sonra şifreyi daha güvenli bir şifre ile değiştirmeniz gerekmekte.

> Bu iki metodda asenkron ve `Task` döndürdüklerinden bunlar `Configure` içerisinde `wait` ile kullanılmalılar. Aksi halde `Configure` metodu çalışmaya devam eder ve bu değişiklikleri atlar. Bunun için normalde `await` kullanmamız gerekmekte. Fakat teknik nedenlerden dolayı `Configure` içerisinde `await` kelimesini kullanamıyoruz. Bundan dolayı başka bir yerde `await` kullanmamız gerekmekte.

Uygulamayı çalıştırdığınızda `admin@todo.local` hesabı oluşturulacak ve `Administrator` rolü atanacak. Bu kullanıcı ile giriş yapaıp `http://localhost:5000/ManageUsers` sayfasına gidin.Uygulamaya kayıt olmuş tüm kullanıcıların olduğu listeyi göreceksiniz.

> Bunun yanında kendiniz yeni bir özellik ekleyebilirsiniz. Örneğin bir buton ile sadece Admin'in kullanıcıları silebileceği değişikliği yapabilirsiniz.

### Görüntü sayfasında kullanıcı kontrolü

`[Authorize]` özelliği ile kontrolör veya aksiyonda kullanıcı kontrolü yapmak mümkün, fakat ya Görüntü dosyasına buna ihtiyacımız olursa? Örneğin "Kullanıcı Yönetimi" bağlantısı menüde olsa çok işimize yarar. Fakat bu linkin sadece Admin kullanıcısına görünmesi gerekmekte.

`UserManager`'i doğrudan Görüntü dosyasına enjekte etmek mümkün, bunun vasıtasıyla istediğiniz kontrolleri yapabiliriz. Görüntü dosyamızı daha temiz tutmak için yeni bir kısmi görüntü dosyası ( partial view ) oluşturun. Bu kısmi görüntü içerisinde kontrollerimizi yapacağız ve eğer giren kullancıcı `Administrator` rolüne sahipse "Kullanıcı Yönetimi" bağlantısını görebilir. 

**`Views/Shared/_AdminActionsPartial.cshtml`**

```html
@using Microsoft.AspNetCore.Identity
@using AspNetCoreTodo.Models

@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager

@if (SignInManager.IsSignedIn(User))
{
    var currentUser = await UserManager.GetUserAsync(User);

    var isAdmin = currentUser != null
        && await UserManager.IsInRoleAsync(currentUser, Constants.AdministratorRole);

    if (isAdmin) {
        <ul class="nav navbar-nav navbar-right">
            <li><a asp-controller="ManageUsers" asp-action="Index">Manage Users</a></li>
        </ul>
    }
}
```
**Kısmi Görüntü** küçük bir görüntü parçasıdır, genelde diğer **Görüntü** dosyalarına gömülürler. Bundan dolayı belirleyici olarak başlangıçlarına `_` işareti koyularak **kısmi** olduğu belirtilir. Tabi bu zorunlu değildir.

Bu kısmi görüntü önce `SignInManager` ile kullanıcının giriş yapıp yapmadığını kontrol eder. Eğer giriş yapmamışlarsa, kodun geri kalanının önemi yoktur. Eğer giriş yapmış kullanıcı varsa, `UserManager` ile `IsInRoleAsync` metodu çağırılarak bu kullanıcının diğer özelliklerine bakılır. Eğer bu kontrol başarılı ise navigasyona "Kullanıcı Yönetimi" eklenir.

Tabi bu kısmi görüntünün ana görüntüye gömülmesi gerekmekte. Bunun için `_Layout.cshtml` içerisine gidin ve aşağıdaki değişikliği yapın.


**`Views/Shared/_Layout.cshtml`**

```html
<div class="navbar-collapse collapse">
    <ul class="nav navbar-nav">
        <li><a asp-area="" asp-controller="Home" asp-action="Index">Home</a></li>
        <li><a asp-area="" asp-controller="Home" asp-action="About">About</a></li>
        <li><a asp-area="" asp-controller="Home" asp-action="Contact">Contact</a></li>
    </ul>
    @await Html.PartialAsync("_LoginPartial")
    @await Html.PartialAsync("_AdminActionsPartial")
</div>
```
Admin kullanıcısı ile girdiğinizde sağ üst tarafta aşağıdaki gibi "Kullanıcı Yönetimi" (Manage Users) bağlantısını göreceksiniz.

![Manage Users link](manage-users.png)

## Özet

ASP.NET Core oldukça güçlü bir güvenlik ve kimlik sistemidir. Bu sistem ile *kimlik doğrulaması* ve *izin* kontrolleri yapabilir. Buna harici kimlik servis sağlayıcılar(facebook, google, twitter) ekleyebilirsiniz. `dotnet new` şablonu size kullanıcı girişi ve kayıdı gibi genel senaryoları içeren projeyi oluşturur. 

ASP.NET Core Kimlik ile ilgili daha detaylı bilgiyi https://docs.asp.net adresinde bulabilirsiniz.
