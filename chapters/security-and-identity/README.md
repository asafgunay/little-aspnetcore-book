# Güvenlik ve Kimlik

Çoğu modern ağ uygulamasında veya Uygulama Program Arayüzü(API) üzerinde en önemli endişelerden biri güvenliktir. Uygulamanızı saldırılara karşı korumanız ve kullanıcıların girdikleri bilgileri güvende tutmanız çok önemlidir. Bu güvenli önemleri şu şekilde sıralanabilir.

* Kullanıcıdan alınan verilerdeki sterilize edip SQL enjeksiyonu ataklarından korunma.
* Cross-Domain (XSRF) saldırılarından korunma
* HTTPS (TLS) kullanarak verinin internette giderken başkalarının araya girmesini engellemek.
* Kullanıcılara güvenli bir giriş ortamı ayarlama. Sosyal ağ hesapları ile de olabilir.
* Şifreyi unuttum veya birçok adımda giriş seçenekleri sunma

ASP.NET Core tüm yukarıdaki işlemleri kolayca yapmanızı sağlar. İlk iki made varsayılan olarak projede bulunmaktadır.(SQL enjeksiyonu ve Cross Domain). HTTPS desteği ise birkaç satır kod ile çözülebilir. Bu bölüm genel olarak **kimlik** üzerinde durmaktadır. Bunlar kullanıcı kaydetme ve giriş yapmasını sağlama. Sonrasında ise bu kullanıcıları yetkilendirebilmedir.

> Yetki ve kimlik doğrulama genel olarak birbirleri ile karıştırılan iki terimdir. **Kimlik doğrulama** kullanıcının giriş yapıp yapamayacağıdır. Buna karşılık **Yetki** ise kullanıcı giriş yaptıktan sonra nelere yetkisi olduğuyla alakalıdır. Şu şekilde düşünebilirsiniz. **Kimlik Doğrulama** "Ben bu kullanıcıyı tanıyor muyum?" diye sorarken. **Yetki** "Bu kullanıcının şunu yapmaya yetkisi var mı?" diye sorar.

projemizi MVC + Kimlik doğrulama şablonu ile oluşturmuştunuz. Bu şablon içerisinde **Kimlik Doğrulama** ve **Yetki** için birçok sınıf doğrudan eklenmiştir.

## ASP.NET Core Kimlik ne işe yarar?
ASP.NET Core Kimlik, ASP.NET Core ile birlikte gelir. ASP.NET CORE ekosisteminde bulunan eklentiler gibi bu da bir NuGet paketidir ve böylece her projeye kurulabilir. ( Eğer varsayılan temayı kullanıyorsanız doğrudan yüklü gelmektedir)

ASP.NET Core Kimlik, kullanıcı bilgilerini tutar şifreleri karıştırarak güvenliğini sağlar ve kullanıcı rollerini yönetir. Email/Şifre bilgileri ile giriş yapılmasını sağlar, sosyal ağ ile kullanıcı girişi yapılmasına zemin hazırlar. Böylece Google, Facebook veya Twitter bilgileri ile kullanıcılar servisinize giriş yapabilir. Bu servisler OAuth 2.0 veya OpenId kullanmaktadırlar.

Kayıt olma ve Giriş ekranları da varsayılan şablon ile gelmektedir. Hatta şu anda kayıt olabilir ve giriş yapabilirsiniz. 
