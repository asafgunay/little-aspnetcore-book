# Uygulamanın Dağıtılması

Uzun bir yol geldiniz, fakat henüz tam anlamıyla bitti diyemeyiz. Artık bu harika uygulamanızı yaptığınıza göre dünya ile paylaşma zamanı geldi demektir.

ASP.NET uygulamaları Windows, Mac veya Linux ortamlarında çalışabildiğinden dolayı bir çok farklı yöntem ile uygulamanızı dağıtabilirsiniz. Bu bölümde en fazla kullanılan yöntemleri göreceksiniz.

## Dağıtma Yöntemleri

ASP.NET Core uygulamaları genel olarak aşağıdaki ortamlardan birisine kurulur.

* **Herhangi bir Docker Hostuna** Docker koşabilecek herhangi bir makineye ASP.NET uygulaması dağıtılabilir. Docker ile daha önceden çalıştıysanız çok hızlı bir şekilde imaj oluşturup uygulamanızı dağıtabilirsiniz.

* **Azure** Microsoft Azure ASP.NET Core uygulamaları için doğrudan destek verir. Eğer Azure servisine kayıtlıysanız, yeni bir web uygulaması oluşturup dosyalarınızı kopyalarsanız projenizi doğrudan çalıştırabilirsiniz. Bunun nasıl yapılacağını ileride Azure CLI üzerinden göreceksiniz.

* **Linux ( Nginx ile)** Eğer Docker ile çalışmak istemiyorsanız, uygulamnızı yine de herhangi bir Linux Sunucusu üzerinden koşabilirsiniz. Bunlar Amazon Ec2 veya DigitalOcean sanal makineleri olabilir. Genelde ASP.NET Core ve Nginx reverse proxy yapılarak kullanılırlar. Aşağıda daha geniş bir şekilde göreceksiniz.


* **Windows** Windows IIS web sunucusunu kullanarak ASP.NET Core uygulaması çalıştırabilirsiniz. Genelde Azure'a kurmak daha kolay ve ucuz yöntem olsa da Windows Sunucularını kendiniz kontrol etmek istiyorsanız bu yöntemi kullanabilirsiniz.


## Kestrel ve reverse proxy.

>  Eğer ASP.NET Core uygulamasını internet ortamına taşımayı istemiyorsanız bundan sonraki bölümleri atlayabilirsiniz.

ASP.NET Core Kestrel adında hızlı ve oldukça hafif bir geliştirme sunucusu kullanır. Projede aslında şimdiye kadar hep bu sunucuyu kullandınız. Uygulamayı kullanılacak(production - live - canlı ) ortama taşıdığınızda yine Kestrel kullanarak çalışabilir. Fakat önerilen yöntem; reverse proxy yaparak Kestrel'in önüne daha yetkin ( load blancing özelliği olan ) bir sunucu kurmaktır.

Linuxta  ve Docker Konteynırında, Nginx veya Apache web server kurarak gelen taleplerin Kestrel'e iletilmesini sağlayabilirsiniz. Windows sunucuda IIS aslında aynı işi yapmaktadır. 

Eğer uygulamanızı Azure üzerinde tutuyorsanız, bunların hepsi otomatik olarak sizin yerinize yapılmaktadır. Nginx ile reverse proxy yapma olayını **Docker ile Dağıtma** bölümünde göreceksiniz.