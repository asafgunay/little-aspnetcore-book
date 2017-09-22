# Ekstra Paket Ekleme

.NET ekosistemini kullanmanın en büyük avantajlarından biriside 3.Parti eklentilerdir. Diğer paket sistemleri gibi ( npm, Maven, RubyGems ) .NET paketleri indirebilir ve kullanabilirsiniz. 

NuGet hem paket yönetim aracı hemde kaynağıdır. ( https://www.nuget.org ) Nuget paketlerini internette aratıp kendi bilgisayarınıza terminal veya komut satırı aracılığıyla veya Visual Studio yardımıyla indirip kurabilirsiniz. Nitekim biz de bu projemizde Humanizer paketini indirip kuracağız

## Humanizer paketi kurma

Bir önceki bölümde yapılacaklar uygulaması ekrana aşağıdaki gibi bir çıktı vermişti : 

![Dates in ISO 8601 format](iso8601.png)

Ekranda görülen tarih bölümü makineler için okunaklı olabilir ( ISO 8601 ), fakat insanlar için okuması zordur. Mesela bunu "X gün kaldı" şekilde yapsak daha okunaklı olmaz mı? Elbette bunu kendimiz kodlayarak yapabiliriz. Daha hızlı yolu ise hazırda yazılmışı ile yapmak,

NuGette bulunan Humanizer paketi(https://www.nuget.org/packages/Humanizer) isminden de belli olduğu gibi insanların okuyabilirliklerini artırma amacıyla yapılmıştır. Bunlar tarih, saat, süreç, sayılar vs. için kullanılır. MIT lisansına sahip harika bir açık kaynak kodlu projedir.

Bu paketi projenize eklemek için aşağıdaki kodu terminalde veya komut satırında çalıştırınız.


```
dotnet add package Humanizer.Core
```

`AspNetCoreTodo.csproj` dosyasını inceleyecek olursanız, `Humanizer.Core` adında yeni bir `PackageReference` göreceksiniz.


## Humanizer'ı Görüntü dosyamızda ( View ) kullanma

Bir paketi kodumuzda kullanabilmek için her zamanki gibi `using` cümlesiyle belirtmemiz gerekmektedir.

Humanizer'i biz kontrolör veya servis katmanında değilde sadece Görüntü katmanında kullanacağımızdan dolayı doğrudan Görüntü dosyasının içerisinde bunu belirtebiliriz.


**`Views/Todo/Index.cshtml`**

```html
@model TodoViewModel;
@using Humanizer;

// ...
```

Daha sonra `DueAt` yazan özelliği `Humanize` metoduyla aşağıdaki gibi yazdığımızda 

```html
<td>@item.DueAt.Humanize()</td>
```

Tarihlerin daha okunabilir olduğunu göreceksiniz.

![Human-readable dates](friendly-dates.png)

Nuget'te XML parçalayan paketten, makine öğrenmesi yapabilen veya twitter'a yeni konu giren pakete kadar her türlü paket bulmak mümkündür. 

> `dotnet new mvc` ile oluşturulan proje dosyası tek bir referans içerir, `Microsoft.AspNetCore.All` paketi. Bu paket aslında diğer referansları tutan bir "metapakettir". Böylece biz tek bir paket indiriyormuşuz gibi dursa da aslında bize gerekli olan tüm paketler arkada indirilir.

Bir sonraki konuda, Entity Framework Core adında yeni bir NuGet paketi göreceğiz. Bu paket bizim veri tabanı ile iletişimimizi sağlayacaktır.
