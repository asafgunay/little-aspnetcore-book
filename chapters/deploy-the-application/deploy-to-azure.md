## Azure'a Dağıtma

ASP.NET uygulamasının Azure'a dağıtılması sadece bir kaç adımdan oluşur. Bu işlem Azure web ortamından yapılacağı gibi, komut satırı üzerinden de yapılabilir. Bunu daha sonra göreceksiniz.


### Ne yapmalıyız

* Komut satırından ( `git --version` kullanarak git'in kurulu olup olmadığını kontrol ediniz.)

* Azure CLI'ı kurun (https://github.com/Azure/azure-cli)
* Azure bedava paketine kayıt olun.
* Dağıtma için projenin ana klasöründeki dosyayı değiştirin.

### Yeni bir dağıtım yapılandırma dosyası oluşturun.

Bizim şu anda 3 tane farklı projemiz olduğundan Azure bunlardan hangisinin dünyaya gösterileceğini bilmemektedir. Bunu düzelmek için bu klasörlerin bulunduğu klasöre `.deployment` adında bir dosya oluştun. 
**`.deployment`**

```ini
[config]
project = AspNetCoreTodo/AspNetCoreTodo.csproj
```
`.deployment` olarak adlandırıldığına emin olur. ( Windows işletim sisteminde dosyayı `".deployment"` şeklinde kaydederek sağlayabilirsiniz.)

Eğer komut satırından `ls` veya `dir`ile bu klasöre giderseniz şunları göreceksiniz:

```
.deployment
AspNetCoreTodo
AspNetCoreTodo.IntegrationTests
AspNetCoreTodo.UnitTests
```

### Azure kaynaklarının ayarlanması

Eğer Azure CLI'ı ilk defa kurduysanız aşağıdaki komutu çalıştırın

```
az login
```
Adrından kullanıcı adı ve şifrenizi yazıp giriş yapın.Sonrasında uygulamamız için yeni bir Kaynak Grubu oluşturun

```
az group create -l westus -n AspNetCoreTodoGroup
```

Sonrasında yeni bir `App Service Plan`(Uygulama Servis Planı) oluşturmanız gerekmekte

```
az appservice plan create -g AspNetCoreTodoGroup -n AspNetCoreTodoPlan --sku F1
```

> `F1` bedava servis planı, eğer kendi domaininizde kullanmak istiyorsanız bunun yerine aylık 10$ vererek D1 planını seçebilirsiniz.

Şimdi Uygulama Servis Planı içerisine web uygulamanızı oluşturun


```
az webapp create -g AspNetCoreTodoGroup -p AspNetCoreTodoPlan -n MyTodoApp
```
Uygulamanın ismi (`MyTodoApp`) Azure içerisinde eşsiz olmalı. Uygulama çalıştığında varsayılan URL bu uygulama ismi olacaktır. Örneğin : http://mytodoapp.azurewebsites.net


### Uygulama ayarlarını güncelleme

> Bu sadece eğer **Güvenlik ve Kimlik** bölümünde Facebook ile giriş işlemini yaptıysanız gerekmektedir.

Uygulamanız `Facebook:AppId` ve `Facebook:AppSecret` ayarları olmadan çalışamayacaktır. Bunları Azure web portalını kullanarak eklemeniz gerekmektedir.

1. Azure hesabı ile https://portal.azure.com sitesine girin.
1. Oluşturduğunuz web uygulamasını açın : `MyTodoApp` 
1. **Application Settings** (Uygulama ayarları) tabına tıklayın.
1. **App settings** bölümün altına `Facebook:AppId` ve `Facebook:AppSecret` değerlerini facebooktan aldığınız değerlere göre değiştirin.
1. Üstte bulunan **Save** butonuna tıklayarak değişiklikleri kaydedin.

### Proje dosyalarının Azure'a dağıtılması

Bunun için Git kullanabilirsiniz. Hala local klasöreleri Gir ile takip edilmiyorsa aşağıdaki komutları yazın. Bu komutları tüm projeleri içeren klasöre içinde yapabilirsiniz.

```
git init
git add .
git commit -m "First commit!"
```

Sonrasında Azure kullanıcı adı ve şifresi oluşturun.

```
az webapp deployment user set --user-name nate
```
Buradaki komutları inceleyerek `config-local-git` ile bir github linki çıktısı almanız lazım.

```
az webapp deployment source config-local-git -g AspNetCoreTodoGroup -n MyTodoApp --out tsv

https://nate@mytodoapp.scm.azurewebsites.net/MyTodoApp.git
```

Bu URL'i kopyalayın ve sonra tekrar projenin ana klasöründe bulunan git reposuna aşağıdaki gibi remote(uzak) olarak gösterin

```
git remote add azure <yapıştır>
```

Bu adımları sadece bir defa yapmanız lazım. Eğer ileride dosyalarınızı tekrar Azure'a göndermek isterseniz. Commit ettikten sonra tek yapmanız gereken
```
git push azure master
```

Önünüze bir çok log mesaji dökülecek ve en sonunda Azure'a dağıtıldığını göreceksiniz. Uygulama adresinize ( http://uygulamaismi.azuerwebsites.net) eriştiğinizde projenizin çalıştığını göreceksiniz.