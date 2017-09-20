## C# ile Merhaba Dünya #

ASP.NET Core a başlamadan önce basit bir C# uygulaması yapıp çalıştıralım

Bunu komut satırı ile yapabilirsiniz. Önce Terminal( veya powershell ) açıp dosyayı kaydetmek istediğiniz yere gidin. Örneğin Documents:

```
cd Documents
```
 `dotnet` komutu ile yeni proje oluşturun:

```
dotnet new console -o CsharpHelloWorld
cd CsharpHelloWorld
```
Bu basit bir C# projesi oluşturacaktır. Oluştururken ekrana bazı şeyler yazabilir. Proje iki önemli dosyadan oluşmaktadır. Bunlar `.csproj` proje dosyası uzantısı ve C# kodu `.cs` uzantısı ile oluşturduğunuz klasörde bulunmaktadır. Eğer editörünüz ile dosyayı açarsanız

**`CsharpHelloWorld.csproj`**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

</Project>
```
Yukarıdaki gibi bir içerik göreceksiniz. Proje dosyası XML tabanlıdır ve proje hakkında bazı üstdata(metadata) göreceksiniz. Daha sonra başka paketlere referans ederseniz bunlar bu dosya içerisinde görünecektir. Yani bu proje npm de bulunan  `project.json` dosyası gibidir. Çoğunlukla bu dosyayı elle değiştirmeniz gerekmemektedir.

**`Program.cs`**

```csharp
using System;

namespace CsharpHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

`static void Main` C# programının giriş noktasıdır. Eğer bu metod bir sınıf içerisinde yazılmışsa buna `Program` denir.  `using` cümlesi sistem içerisindeki sınıfları getirmek için kullanılır. Bunlar .NET sınıfları olabilecei gibi başka sınıflar da olabilir.

Projenin içerisinde terminalinizden(Komut satırı)  `dotnet run` yazarak programı çalıştırabilirsiniz. Programınız çalıştığında aşağıdaki çıktıyı göreceksiniz.

```
dotnet run

Hello World!
```
Bu bölümde .NET projesinin nasıl yaratılıp çalıştırıldığını gördünüz. Artık ASP.NET Core uygulamasına geçebiliriz.