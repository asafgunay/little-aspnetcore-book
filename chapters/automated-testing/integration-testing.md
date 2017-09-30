## Entegrasyon Testi

Unit teste nazaran entegrasyon testi yönlendirme, kontrolörler, servisler ve veri tabanlarının tamamını test etmeyi amaçlar. Bir sınıfı izole etmek yerine tüm komponentler düzgün bir şekilde senkronize çalışıyor mu bunun kontrol edilmesini sağlar.

Entegrasyon testleri daha yavaş ve unit testlere göre daha agrasif yani herşeyi test etme amacındadır. Bir projenin bir çok unite testi olsa da entegrasyon desti oldukça azdır.


Yönlendirme de dahil herşeyin test edilmesi gerektiğini savunduğundan, genelde web tarayıcı gibi HTTP çağrısı yapabilecek yapılar içerir.

Entegresyon testini yapabilmek için sunucunuzun çalışır durumda olması gerekir. ASP.NET Core test etme amacıyla bu sunucunun `TestServer` sınıfıyla çalıştırılıp test bittikten sonra otomatik olarak kapatılmasını sağlayabilir.

### Yeni bir test projesi oluşturun

Entegrasyon testi ile unite testini birlikte aynı projede çalıştırabilirsiniz. Fakat daha güzenli olması açısından farklı projeler oluşturmak daha mantıklı.

Eğer proje dizininde iseniz  bir üst klasöre `cd` ile çıkın ve şimdi `AspNetCoreTodo` klasöründe olmanız gerekmekte. Aşağıdaki komutlar ile yeni bir proje oluşturun.


```
mkdir AspNetCoreTodo.IntegrationTests
cd AspNetCoreTodo.IntegrationTests
dotnet new xunit
```

Klasör yapınız aşağıdaki gibi olmalı:


```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj

    AspNetCoreTodo.IntegrationTests/
        AspNetCoreTodo.IntegrationTests.csproj
```

Unit testte yaptığımız gibi ana projeye referans vermemiz gerekmekte.

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```
Ayrıca `Microsoft.AspNetCore.TestHost` NuGet paketini de aşağıdaki gibi ekleyin:


```
dotnet add package Microsoft.AspNetCore.TestHost
```

En son olarak `UnitTest1.cs` dosyasını silebilirsiniz. Artık entegrasyon testi yazmaya hazırsınız.

### Write an integration test

Her testten önce sunucu üzerinde bazı ayarlamaların yapılması gerekmekte. Bunu test kurulum sınıfı yerine farklı bir sınıfta gerçekleştirerek kurulum sınıfını daha temiz bırakabiliriz. `TestFixture` adında yeni bir sınıf oluşturun


**`AspNetCoreTodo.IntegrationTests/TestFixture.cs`**

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Configuration;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TestFixture : IDisposable  
    {
        private readonly TestServer _server;

        public TestFixture()
        {
            var builder = new WebHostBuilder()
                .UseStartup<AspNetCoreTodo.Startup>()
                .ConfigureAppConfiguration((context, configBuilder) =>
                {
                    configBuilder.SetBasePath(Path.Combine(
                        Directory.GetCurrentDirectory(), "..\\..\\..\\..\\AspNetCoreTodo"));

                    configBuilder.AddJsonFile("appsettings.json");

                    // Add fake configuration for Facebook middleware (to avoid startup errors)
                    configBuilder.AddInMemoryCollection(new Dictionary<string, string>()
                    {
                        ["Facebook:AppId"] = "fake-app-id",
                        ["Facebook:AppSecret"] = "fake-app-secret"
                    });
                });
            _server = new TestServer(builder);

            Client = _server.CreateClient();
            Client.BaseAddress = new Uri("http://localhost:5000");
        }

        public HttpClient Client { get; }

        public void Dispose()
        {
            Client.Dispose();
            _server.Dispose();
        }
    }
}
```
Bu sınıf `TestServer`'ın kurulmasını ve test yapılacak sınıfın bu şekilde sunucu kurulum kodlarından ayrı olmasını sağlar.

> **Güvenlik ve Kimlik** bölümünde Facebook ile girişi ayarladıysanız, Facebook app ID ve secret için yukarıdaki kodda `ConfigureAppConfiguration` bölümünde sahte veriler girmelisiniz. Aksi halde Gizlilik Yönetici'de bu değerler olmadığından sunucunuz başlamayacaktır.


Artık entegrasyon testi yazmaya hazırsınız. `TodoRouteShould` adında yeni bir sınıf oluşturun


**`AspNetCoreTodo.IntegrationTests/TodoRouteShould.cs`**

```csharp
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Xunit;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TodoRouteShould : IClassFixture<TestFixture>
    {
        private readonly HttpClient _client;

        public TodoRouteShould(TestFixture fixture)
        {
            _client = fixture.Client;
        }

        [Fact]
        public async Task ChallengeAnonymousUser()
        {
            // Ayarla
            var request = new HttpRequestMessage(HttpMethod.Get, "/todo");

            // Cagir
            var response = await _client.SendAsync(request);

            // kontrol et
            Assert.Equal(HttpStatusCode.Redirect, response.StatusCode);
            Assert.Equal("http://localhost:5000/Account/Login?ReturnUrl=%2Ftodo",
                        response.Headers.Location.ToString());
        }
    }
}
```
Bu test kullanıcı girişi yapmamış birisinin `/todo` yoluna talep gönderdiğinde geri gelen cevabı kontrol eder. Bu durumda giriş ekranına yönlendirmesi gerekmekte.

Bu senaryo entegrasyon testi için iyi bir örnektir. Yönlendirmeleri göz önüne alır, kontrölörlerde bulunan `[Authorize]` sınıflarını kontrol eder. Böylece birisi `[Authorize]` özelliğini kontrolörden silerse hata döner.

Testi terminalde, bir önceki bölümde olduğu gibi,  `dotnet test` komutu ile çalıştırın

```
Starting test execution, please wait...
[xUnit.net 00:00:00.7237031]   Discovering: AspNetCoreTodo.IntegrationTests
[xUnit.net 00:00:00.8118035]   Discovered:  AspNetCoreTodo.IntegrationTests
[xUnit.net 00:00:00.8779059]   Starting:    AspNetCoreTodo.IntegrationTests
[xUnit.net 00:00:01.5828576]   Finished:    AspNetCoreTodo.IntegrationTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 2.0588 Seconds
```


## Özet

Test çok geniş bir alan ve öğrenilecek çok şey var. Örneğin bu bölümde Ön-yüz tesine hiç girmedik, bu konuyu tek başına kitap olacak kadar geniştir. Fakat testin nasıl yapılması gerektiğine dair bir fikriniz var. Bunun için kendinizi geliştirebilir ve daha fazla örnek yapabilirsiniz.

Her zamanki gibi, ASP.NET Core dökümanı en iyi yardımcınız olacaktır. Bunun yanında StackOverflow bir sorun ile karşılaştığınızda başvurabileceğiniz iyi bir kaynaktır.
