# Giriş

Ufak ASP.NET kitabına başladığınız için teşekkürler. Bu kısa
kitabı yazmamın amacı ASP.NET Core 2.0 hakkında bilgi almak isteyen yazılımcılara kolay bir başlangıç yapmalarını sağlamak. Bu kitapta yeni framework ve bu framework ile nasıl web uygulamaları yapılacağını göreceksiniz.

Bu kitap aşağıdaki yapıda hazırlanmıştır. Bu projede bir web to-do uygulaması yapacağız:

* MVC patern temeli ( Model-View-Controller) 
The basics of the MVC (Model-View-Controller) pattern
* Önyüz(Front-end) ile Arka uç(Back-end) birlikte nasıl çalışır.
* Dependency injection(Bağımlılık Enjeksiyonu) nasıl yapılır ve neden kullanışlıdır.
* Veri tabanına nasıl yazılır? Veri tabanından nasıl okunur?
* Giriş, kayıt ve güvenlik nasıl yapılır.
* Yaptığımız projeyi nasıl web ortamına taşırız.

Bu projeyi tamamlamak için ASP.NET core ile alakalı hiç birşey bilmek zorunda değilsiniz.

## Başlamadan Önce
Bu projenin tamamlanmış halin GitHub'da(https://www.github.com/nbarbettini/little-aspnetcore-todo) mevcut, bilgisayarınıza indirip kendi versiyonunuz ile kıyaslayabilirsiniz. GitHub . 

Bu kitabın ingilizcesine buradan(https://github.com/nbarbettini/little-aspnetcore-book) erişebilirsiniz. Ben çeviri işlemlerini yapacağım ve bunu Türkçe kitap olarak web ortamında yayınlayacağım.

Bu kitap problemler düzeltildikçe yenilenmektedir. İngilizce yeni versiyonuna ([littleasp.net/book](http://www.littleasp.net/book)) 'dan ulaşabilirsiniz. Kitabın en son sayfasına bakarak versiyonuna ve değişen bölümlere ulaşabilirsiniz.

## Bu kitap kimin için ?

Eğer yazılım ile uğraşıyorsanız bu kitap size modern web uygulamarının nasıl yapılacağına dair bir giriş niteliğindedir. Web uygulamalarının nasıl yapılacağını 0'dan öğreneceksiniz. Doğal olarak küçük bir kitap olduğundan bunun içerisinde herşeyi bulamayabilirsiniz. Fakat bu daha ileri uygulamalar için bir başlangıç niteliğindedir.

Eğer ASP.NET MVC geliştiriciyseniz, lütfen evinizdeymiş gibi hissedin. ASP.NET Core sizin bildiklerinize ek olarak yeni araçlar ve basitlikler getirmekte. Aşağıda bazı değişikliklere değineceğim.

Daha önce hangi dilde çalıştığınıza bağlı olmaksızın bu kitap size basit bir ASP.NET uygulamasının nasıl yapılacağını gösterecektir. Bu kitap ile Ön-yüz ve Arka-uç'u nasıl yapılandıracağınızı, birbirleriyle iletişimi nasıl kuracağınız, database'e nasıl bağlanacağınızı, nasıl test edeceğinizi ve web ortamına nasıl taşıyacağınızı öğreneceksiniz.

## ASP.NET CORE nedir?
ASP.NET Core Microsoft tarafından web uygulamaları inşa etmek için yapılmış bir iskelettir(framework). Bu iskelet ile APIler ve Microservisler oluşturulabilir. İskelet çokça bilinen MVC(Model-View-Controller), bağımlılık enjeksiyonu gibi paternleri kullanmaktadır. Apache 2.0 lisansına sahiptir ki bu da kaynağın ulaşılabilir ve bedava olduğunu gösterir. 

ASP.NET Core Microsof .NET runtime üzerine kuruludur. Bu Java Virtual Machine benzeridir. Böylece birçok dil ile ASP.NET Core ugulamaları yazabilirsiniz. Örneğin C#, Visual Basic, F# gibi. C# en popüleri olduğundan dolayı bu kitapta C# kullanılacaktır. Bunun yanında ASP.Net Core ugulamasını Windows, Mac ve Linux işletim sistemlerinde kullanabilirsiniz.

## Neden başka bir web isketine ihtiyacımız var ?
Hali hazırda bir çok web iskeleti bulunmakta. Örneğin Node/Express, Spring, Ruby on Rails, Django, Laravel ve daha bir çoğu. ASP.Net Core'un bu iskeletlere üstünlüğü nedir ? 

* **Hız.** ASP.NET Core hızlıdır. .NET kodu derlenmiş olduğundan dolayı Interpreted(yorumlanmış) dillere göre ( Javascript, Ruby ) daha hızlıdır. Ayrıca ASP.NET multitreading ve asenkron işler(tasks) için en uygun hale getirilmiştir. Bundan dolayı Node.js'e göre 5-10 kat arası daha hızlıdır.

* **Ecosystem.**  ASP.NET Core belki yeni olabilir fakat .NET çok uzun süredir kullanılmaktadir. Bundan olayı binlerce paket Nuget ( paket yönetimi ) vasıtasıyla kullanılabilmektedir. Nuget npm, Ruby gems veya Maven tarzı bir paket yönetim uygulamasıdır. Örneğin şu anda JSON deserialization, database bağlayıcıları, PDF oluşturma araçları ve binlercesi.

* **Güvenlik.** 
Microsoft takımın güvenliği oldukça ciddiye almaktadır. ASP.NET core baştan sona güvenlik üzerine kurulmuştur. Örneğin input sterilizasyonu veya cross-site request forger(XSRF) otomatik olarak çalışmaktadır. Böylece sizin bunları kontrol etmenize gerek yoktur. Ayrıca Static Typing olduğundan dolayı .NET derleyicinin tüm özelliklerinden faydalanabilirsiniz. Static Typing için tüm yapının düzgün olmasını sağlamaktadır. Böylece data yapısı üzerinde istenilmeyen şeyler yaptığınızda doğrudan hata alacaksınız.

## .NET Core ve .NET Standart
Bu kitapta ASP.NET core (web iskeleti) öğreneceksiniz. Bazen .NET runtime terimini duyabilirsiniz. Ayrıca bu kitap boyunca .NET Core ve .NET Standart terimlerini duyabilirsiniz. Bu kafanızda bir karışıklığa neden olabilir. Kısaca şu şekilde anlatmaya çalışayım:

**.NET Standard** Platform bağımsız .NET'te bulunan API leri tanımlar. .NET standart kod veya fonksiyonaliteye dair birşey söylememektedir. Sadece API tanımıdır. Birçok çeşit "versiyon" veya "kadame" .NET Standart bulunmakta. Bu Standart kaç tane API olduğundan bahsetmektedir. Örneğin .NET Standar 2.0 .NET Standart 1.5'e göre daha fazla API içermektedir.

**.NET Core** ise Windows, Mac veya Linux'a kurulan .NET runtime'dır. .NET standard'da tanımlanan interfaceleri burada anlamlandırırsınız(implement) Tabi bu anlamlandırma platform tabanlıdır. Böylece siz .NET Core'u kendi platformunuza göre kurarsınız. Örneğin **.NET Framework** .NET Standard'ın sadece Windows için anlamlandırdığı(implement) iskelettir. .NET Core çıkana kadar yalnızca .NET Framework vardı.

Eğer isimlendirmekte kafanız karışıyorsa çok takılmayın bu konuya. Projede daha iyi anlayacaksınız.

## ASP.NET 4 geliştiricileri için not 

Eğer daha önce ASP.NET kullanmadıysanız lütfen bir sonraki bölüme geçiniz!

ASP.NET Core tamamen ASP.NET'in baştan yazılmış halidir, yeni nesil teknolojilere daha iyi ayak uydurabilmek ve System.Web'den, IIS ve Windows'tan ayırabilmek için yapımıştır. ASP.NET 4 de  OWIN/Kata olaylarını hatırlarsınız. Bu Katana projesi ASP.NET 5 oldu ve buna ASP.NET Core dediler

Katana projesine göre `Startup` sınıfı öne alındı ve artık 
`Application_Start` veya `Global.asax` kullanmamıza gerek kalmadı.Tüm iletişim hattı middleware üzerinden sağlanıyor ve artık MVC ve WEB API ayrımı kalktı bu da demek oluyor ki Controllerdan doğrudan View veya status code veya data döndürebiliyoruz. Bağımlılık Enjeksiyonu ( Dependency Injection) doğrudan geliyor böylece hiç bir StructureMap veya Niject tarzı container yüklemenize gerek kalmadı. Ayrıca tüm framework hız ve runtime verimi üzerinde iyileştirildi.

Bu kadar giriş bölümü yeterli. Şimdi ASP.NET Core'un derinliklerine inebiliriz.

NOT: Türkçeleştirme ile alakalı yanlış yaptığım veya sizin gözünüze çarpan bir sorun var ise lütfen yeni bir github konusu(issue) açın.