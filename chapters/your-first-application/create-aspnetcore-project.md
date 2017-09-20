## ASP.NET Projesi Oluşturma
Hala Merhaba Dünya( Hello World ) projesi içerisindeyseniz. Bir üst klasöre geçiniz

```
cd ..
```
Daha sonra yeni bir proje oluşturmak için  `dotnet new`, ve birkaç parametre kullanacağız

```
dotnet new mvc --auth Individual -o AspNetCoreTodo
cd AspNetCoreTodo
```

Bu işlem  `mvc` temalı yeni bir proje oluşturacak ve buna ek olarak kimlik doğrulama için güvenlik yapısını projemize ekleyecektir. Bunu daha sonra *Güvenlik* bölümünde daha geniş biçimde inceleyeceğiz.

Klasöre girdiğinize içinde birkaç dosya ve klasöre göreceksiniz. Şu anda yapmanız gereken ilk şey projeyi çalıştırmak ve aşağıdaki çıktıyı almak.

```
dotnet run

Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

Program bitmedi dikkat ederseniz bunu yerine 5000 portundan çalışan bir web sunucu başlattı ve gelecek talepleri dinlemeye başladı.

Tarayıcınızı açıp `http://localhost:5000` adresine gittiğinde ASP.NET başlangıç sayfasını göreceksiniz. Artık projenizin çalıştığına eminsiniz. Terminalde Ctrl-C tuş kombinasyonu ile sunucuyu durdurabilirsiniz

### ASP.NET Core projesinin parçaları

 `dotnet new mvc` teması sizin için birçok dosya ve klasör oluşturur. Bunlar içerinde en önemlileri aşağıdaki gibidir:

* **Program.cs** ve **Startup.cs** 
Dosyaları web server ve ASP.NET Core bağlantılarının oluşturulmasını sağlar. `Startup` sınıfı ek olarak özelkatman(middleware) eklemenizi sağlar ve bu şekilde statik içerik sunabilir veya hata sayfalarını kontrol edebilirsiniz. Ayrıca bağımlı enjeksion taşıyıcıya kendi servisinizi ekleyebilirsiniz(Bu konuya daha sonra geleceğiz). 

* **Models**, **Views** ve **Controllers** Klasörleri Model-View-Controller mimarisine has bileşenleri tutar. Bu üç klasörü bir sonraki konuda işleyeceğiz.

* **wwwroot** Klasörü statik dosyaları tutar. Bunlar CSS, JavaScript, ve imaj dosyalarıdır. Başlangıçta  CSS ve Javascript paketleri bower ile yönetilir. Fakat bunun yerine başka paket yönetilicileri (nmp veya yarn) da kullanmak mümkündür. `wwwroot` klasöründeki dosyalar statik olarak sunulacaktır. Ayrıca bu içerikler otomatik olarak minimize edilebilir.

* The **appsettings.json** dosyası ASP.NET Core'un başlangıçta kullanacağı dosyadır. Burada database bağlantıları ve proje içerisinde doğrudan yazmak istemediğiniz ( hard coded ) şeyleri burada tanımlayabilirsiniz.

### Visual Studio Code için ipuçları

Eğer Visual Studio Code veya Visual Studio kullanıyorsanız, bir kaç ipucu vermek isterim.

* **F5 ile proje çalıştırma ve hata yakalama yakalama**: 

Projeniz açıkken F5'e basıp projenizi çalıştırabilirsiniz. Bu komut satırından yazdığınız `dotnet run` komutu ile aynıdır. Bunun yanında hata yakalamak için ara verme noktası ( breakpoint) koyarak daha etkin bir şekilde çalışabilirsiniz. Bunun için göreselde olduğu gibi sol tarafta bulunan boşluğa tıklamanız gerekmektedir.

![Breakpoint in Visual Studio Code](breakpoint.png)

* **Ampül butonu ile problem çözümü**: Eğer projenizde derleme hatası alırsanız ( kırmızı alt çizgiler) bu kodun üzerine tıklayıp sol tarafına bakın. Size hataya dair bir öneri verecek veya düzeltmenize yardımcı olacaktır. Örneğin  size eksik `using` cümlesini gösterecek ve bunu otomatik olarak getirtmek için yardımcı olacaktır.Ampül Örneği:

![Lightbulb suggestions](lightbulb.png)



* **Kolayca derle**: `Command-Shift-B` veya `Control-Shift-B` ile  `dotnet build` komutunu çalıştırabilirsiniz. Böylece komut satırından birşey yazmanıza gerek kalmaz.

### Git hakkında not

Eğer kodunuzu Git veya Github üzerinden yönetiyorsanız şimdi  `git init` komutunu çalıştırarak projeye git ile başlamanızın zamanı. `.gitignore` dosyasını ekleyerek  `bin` ve `obj` gibi Git'e eklenmesine gerek olmayan dosyaları sistemden çıkarabilirsiniz.Github üzerindeki Visual Studio gitignore iskeleti olduka iyi çalışmaktadır.(https://github.com/github/gitignore).

Daha üzerinde durulacak birçok konu var. Artık projemize geçebiliriz ve uygulamamızı yazabiliriz.