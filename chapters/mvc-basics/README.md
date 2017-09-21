# MVC Temelleri

Bu bölümde ASP.NET Core MVC sistemini inceleyeceğiz. **MVC** (Model-View-Controller) web uygulamaları geliştirmek için kullanacağımız ve neredeyse tüm web iskeletlerinde kullanılan ( Ruby on Rails ve Express ) bir kalıptır. Ayrıca Angular gibi ön yüz iskeletlerinde de kullanılmaktadır. IOS ve Android üzerinde de yine uygulama yazmak için MVC benzeri bir yapı kullanılmaktadır.

İsminde belirtildiği gibi MVC üç bileşenden oluşmaktadır bunlar : modeller, view(görüntüler) ve kontrolörler. **Kontrolörler**  web browser üzerinden gelen talepleri karşılayarak hangi kod parçacığının çalışacağına karar verir. **Views ( görüntüler)** genelde sadece tema görevini görür bunlar genelde html şablon dilleridir. Örneğin Handlebars, Pug veya Razor. Yaptığı işlem sadece kontrolörden gönderilen veriyi almak ve ekranda göstermektir. **Modeller** verilerin tutulmasını ve Görüntülere eklenerek kullanıcıya gösterilmesini sağlar.

MVC için genel kalıp şu şekildedir:

* Kontrolör talebi alır ve veritabanında bulunan bilgilere bakar.
* Kontrolör bu bilgilerden model üreterek Görüntü(view)'e ekler.
* Görüntü kullanıcının tarayıcısında görünür.
* Kullanıcı butona tıklar ve yeni bir talebi kontrolöre gönderir.

Eğer daha önce MVC kalıbı ile çalıştıysanız, kendinizi evinizdeymiş gibi rahat hissedeceksiniz. Eğer MVC ile çalışmadıysanız bu bölüm basit bir şekilde başlamınıza yardımcı olacaktır.

## Ne Yapacağız 

MVC üzerinde yapılan "Merhaba Dünya" uygulaması "Yapılacaklar Listesi" uygulamasıdır. Küçük ve basit bir projedir fakat MVC'nin tüm bölümlerini kullanmanız gerekmektedir. Bu da büyük uygulamalarda kullanacağınız çoğu konsepti kapsamaktadır.

Bu kitapta yapılacaklar listesi uygulaması yapacağız. Bu listeye kullanıcı yeni madde ekleyebilecek veya bittiğinde onay kutusu ile tamamlandığını belirtebilecek. Bunun için Arka uçta ASP.NET Core, C# ve MVC kalıbı kullanacağız. Önyüzde ise HTML, CSS ve Javascript kullanacağız.

Eğer `dotnet new mvc` ile hala projenizi oluşturmadıysanız lütfen önceki bölüme bakarak projeyi oluşturun ve sayfanın ulaşılabilir olduğunu tarayıcınızdan konrol edin.
